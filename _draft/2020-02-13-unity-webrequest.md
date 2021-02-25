```c#
public async UniTask LoadAdvertisementInfo()
{
	var client = new HttpClient();
	
	using( var requestMessage = new HttpRequestMessage(HttpMethod.Get, _uri))
	using(var response = await client.SendAsync(requestMessage))
	{
		var text = await response.Content.ReadAsStringAsync();
		AdvertisementInfo = adInfoConverter.Convert(text);
	}
}
```





```c#
public static IEnumerator Await(this AsyncOperation operation) 
{
    while(!operation.isDone)
        yield return operation;
}

yield return SceneManager.LoadLevelAsync("blaaaah").Await();
```

