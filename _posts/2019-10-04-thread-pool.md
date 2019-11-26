---
layout: post
title : "Thread Pool"
categories : devlog
tags : c++ thread pool
toc : true
typora-root-url : ..
---



# 스레드풀 소스 코드

```c++
#include <chrono>
#include <condition_variable>
#include <cstdio>
#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>

class ThreadPool
{
	enum class ThreadState
	{
		NONE = -1,
		TERMINATE = 0,
		WAIT_PAUSED = 1,
		PAUSED = 2,
		SLEEPING = 3,
		BUSY = 4,
		AWAITING = 5,
	};

	using AsyncQueue = std::queue<std::function<void()>>;
public:

	ThreadPool();
	virtual ~ThreadPool()
	{
		if (m_state != ThreadState::TERMINATE)
		{
			Stop();
		}
	}

	void Initialize(int init_thd)
	{
		m_workers.reserve(init_thd);

		m_state = ThreadState::BUSY;

		for (auto i = 0; i < init_thd; ++i)
		{
			m_workers.emplace_back([this]() { this->WorkerUpdate(); });
		}
	}
	void Stop()
	{
		m_state = ThreadState::TERMINATE;
		m_cond.notify_all();

		for (auto &w : m_workers)
		{
			w.join();
		}
	}
	// 모든 워커 스레드를 일시정시 시킨다.
	void PauseWorker()
	{
		m_state = ThreadState::PAUSED;
		m_cond.notify_all();
	}

	// 모든 워커 스레드를 재가동 한다.
	void ResumeWorker()
	{
		m_state = ThreadState::BUSY;
	}

	void WorkerUpdate()
	{
		while (m_state != ThreadState::TERMINATE)
		{

			if (m_state == ThreadState::PAUSED)
			{
				while (m_state == ThreadState::PAUSED)
				{
					std::this_thread::sleep_for(std::chrono::milliseconds(200));
				}
			}

			std::unique_lock<std::mutex> lock(m_job_lock);

			m_cond.wait(lock, [this]() {return !this->m_jobs.empty() || m_state == ThreadState::TERMINATE || m_state == ThreadState::PAUSED; });

			if (m_state == ThreadState::PAUSED)
				continue;

			if (m_state == ThreadState::TERMINATE)
				break;

			auto job = std::move(m_jobs.front());
			m_jobs.pop();
			lock.unlock();

			std::chrono::system_clock::time_point begin_time = std::chrono::system_clock::now();

			job();

			auto diff = std::chrono::duration_cast<std::chrono::milliseconds>(std::chrono::system_clock::now() - begin_time).count();

		}
	}
	ThreadState GetState() { return m_state; }

	template< class F, class ... Args>
	std::future<std::invoke_result_t<F, Args...>> InsertToThread(F f, Args... args);

private:
	std::atomic<ThreadState> m_state = ThreadState::NONE;
	std::condition_variable m_cond;
	std::vector<std::thread> m_workers;
	AsyncQueue m_jobs;
	std::mutex m_job_lock;
};

template< class F, class ... Args>
std::future<std::invoke_result_t<F, Args...>>
ThreadPool::InsertToThread(F f, Args... args)
{
	if (m_state == ThreadState::TERMINATE)
	{
		throw std::runtime_error("ThreadPool was terminated.");
	}

	using return_type = std::invoke_result_t<F, Args...>;

	auto job = std::make_shared<std::packaged_task<return_type()>>(std::bind(f, args...));

	std::future<return_type> job_result_future = job->get_future();
	{
		std::lock_guard<std::mutex> lock(m_job_lock);
		m_jobs.push([job]() { (*job)(); });
	}
	m_cond.notify_one();

	return job_result_future;

}

// 사용 예시
int work(int t, int id) {
	printf("%d start \n", id);
	std::this_thread::sleep_for(std::chrono::seconds(t));
	printf("%d end after %ds\n", id, t);
	return t + id;
}

int main() {
	ThreadPool pool(3);

	pool.Initialize(3);

	std::vector<std::future<int>> futures;
	for (int i = 0; i < 10; i++) {
		futures.InsertToThread(pool.EnqueueJob(work, i % 3 + 1, i));
	}
	for (auto& f : futures) {
		printf("result : %d \n", f.get());
	}
}
```

