---
layout: post
title: "IDisposable接口与ReaderWriterLock结合使用技巧"
date: 2017-05-13   
tag: 架构设计 
---

### 使用IDisposable与ReaderWriterLock 可以很好的解决共享资源的并发问题

```csharp

//具体实例
public abstract class RouteBase
{
	public abstract RouteData GetRouteData(HttpContextBase httpContext);

	public abstract VirtualPathData GetVirtualPath(RequestContext requestContext, RouteValueDictionary values);
}

public class RouteCollection : Collection<RouteBase>
{
	//读锁
	private class ReadLockDisposable : IDisposable
	{
		private ReaderWriterLock _rwLock;

		public ReadLockDisposable(ReaderWriterLock rwLock)
		{
			this._rwLock = rwLock;
		}

	    public void IDisposable.Dispose()
		{
			this._rwLock.ReleaseReaderLock();
		}
	}
	//写锁
	private class WriteLockDisposable : IDisposable
	{
		private ReaderWriterLock _rwLock;

		public WriteLockDisposable(ReaderWriterLock rwLock)
		{
			this._rwLock = rwLock;
		}

		public	void IDisposable.Dispose()
		{
			this._rwLock.ReleaseWriterLock();
		}
	}
}
```