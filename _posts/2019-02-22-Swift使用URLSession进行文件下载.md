---
layout: post
title: "Swift使用URLSession进行文件下载"
categories: IOS
date: 2019-02-22 20:18:00
---

总体的内容分两部分：
* 使用URLSession下载一张图片
* 多文件下载的下载管理器

### URLSession的使用
首先是创建一个请求：

```swift
 //创建请求
var urlRequest = URLRequest(url: URL(string: url)!)
urlRequest.httpMethod = "GET"//下载图片用GET请求
```
然后创建URLSession，session创建需要传入3个参数。
* 第一个是环境配置，默认的default配置是会持久化存储一些缓存、cookie、证书等等信息
* 第二个是代理，异步返回网络请求状态、数据
* 第三个是线程队列，默认会新启一个子线程

```swift
// session会话
session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
```
最后开始启动下载任务：

```swift
// 创建下载任务
let downloadTask = session?.dataTask(with: urlRequest)
// 开始下载
downloadTask?.resume()
```


<!-- more -->



下面是网络请求代理的实现，因为是下载图片所有这里用了`URLSessionDataDelegate`代理，完整的实现代码：

```swift
    
    // 接收到服务器响应的时候调用该方法 completionHandler .allow 继续接收数据
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse, completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
        print("开始响应...............")
        completionHandler(.allow)
    }
    
    //接收到数据 可能调用多次
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        print("接收到数据...............")
        if self.responseData == nil {
            self.responseData = Data.init()
        }
        // 因为多次返回数据，这里把data合并
        self.responseData?.append(data)
        // 已经接收到的数据大小
        let currentLength = Float(self.responseData?.count ?? 0)
        // 这里有个问题 有些写的不规范的 header里面没有length 那就无法计算进度
        let totalLength = Float(dataTask.response?.expectedContentLength ?? -1)
        var progress = currentLength / totalLength
        if totalLength<0 {
            progress = 0.0
        }
        print("current: \(currentLength) , total:\(totalLength), progress:\(progress)")
        // 刷新进度条
        weak var downloadSelf = self
        DispatchQueue.main.async {
            downloadSelf?.refreshUI(progress: progress, error: nil)
        }
    }
    //下载结束 error有值表示失败
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        print("下载完成")
        weak var downloadSelf = self
        DispatchQueue.main.async {
            downloadSelf?.refreshUI(progress: 1.0, error: error)
            // 下载完成要释放资源
            downloadSelf?.session?.invalidateAndCancel()
            downloadSelf?.session = nil
            downloadSelf?.responseData  = nil
        }
    }
```
![URLSession下载.2019-02-22 21_25_00](http://img.muliba.net/URLSession%E4%B8%8B%E8%BD%BD.2019-02-22%2021_25_00.gif)

全部代码在：[URLSessionDownloadViewController.swift](https://github.com/fancylou/SwiftURLSessionDownload)

### 文件下载管理、同时下载多个文件
上面说的是前台一个文件下载的过程，有时候业务需求会同时下载多个文件的情况。这时候需要一个管理多个下载任务的功能类：

```swift

class DownloaderManager: NSObject {
    // 单例模式
    static let shared : DownloaderManager = DownloaderManager()
    // 下载缓存池
    private var downloadCache: Dictionary<String, Downloader>
    
    private override init() {
        downloadCache = Dictionary()
    }
    
    
    func download(url: URL, progress: @escaping DownloadProgressClosure, completion: @escaping DownloadCompletionClosure, fail: @escaping DownloadFailClosure)  {
        self.failback = fail
        // 判断缓存池中是否已经存在
        var downloader = self.downloadCache[url.path]
        if downloader != nil {
            print("当前下载已存在不需要重复下载！")
            self.failback?("当前下载已存在不需要重复下载！")
            return
        }
        downloader = Downloader()
        self.downloadCache[url.path] = downloader
        weak var managerWeak = self
        downloader?.download(url: url, progress: progress, completion: { (filePath) in
            managerWeak?.downloadCache.removeValue(forKey: url.path)
            completion(filePath)
        }, fail: fail)
    }
    
    func cancelTask(url: URL) {
        let downloader = self.downloadCache[url.path]
        if downloader == nil {
            self.failback?("任务已经移除，不需要重复移除！")
            return
        }
        //结束任务
        downloader?.cancel()
        // 从缓存池中删除
        self.downloadCache.removeValue(forKey: url.path)
    }
}
```
这个管理器管理多个下载任务`Downloader`， 每个`Downloader`其实就是上面单个图片下载的代码提取出来。
Downloader：

```swift
//开始下载
func download(url: URL, progress: @escaping DownloadProgressClosure, completion: @escaping DownloadCompletionClosure, fail: @escaping DownloadFailClosure) {
        self.progress = progress
        self.completion = completion
        self.fail = fail
        self.downloadUrl = url
        
        guard self.downloadSession == nil else {
            print("已经开始下载。。。。")
            return
        }
        //开启子线程进行下载任务
        DispatchQueue.global().async {
            var request = URLRequest(url: self.downloadUrl!)
            request.httpMethod = "GET"
            request.cachePolicy = .reloadIgnoringLocalCacheData
            // session会话
            self.downloadSession = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
            // 创建下载任务
            let downloadTask = self.downloadSession?.dataTask(with: request)
            // 开始下载
            downloadTask?.resume()
            // 当前运行循环
            self.downloadLoop = CFRunLoopGetCurrent()
            CFRunLoopRun()
        }
        
    }
    
    // 取消下载任务
    func cancel() {
        self.downloadSession?.invalidateAndCancel()
        self.downloadSession = nil
        self.fileOutputStream = nil
        // 结束下载的线程
        CFRunLoopStop(self.downloadLoop)
    }
```
接收数据的代理：

```swift
extension Downloader: URLSessionDataDelegate {
    // 接收到服务器响应的时候调用该方法 completionHandler .allow 继续接收数据
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse, completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
        print("远程服务器开始响应...............")
        //初始化本地文件地址
        // 远程文件名称
        let filename = dataTask.response?.suggestedFilename ?? "unKnownFileTitle.tmp"
        let dir = FileManager.default.urls(for: .downloadsDirectory, in: .userDomainMask)[0]
        localUrl = dir.appendingPathComponent(filename)
        print("本地地址：\(self.localUrl?.absoluteString ?? "")")
        self.fileOutputStream = OutputStream(url: self.localUrl!, append: true)
        self.fileOutputStream?.open()
        completionHandler(.allow)
        
    }
    
    //接收到数据 可能调用多次
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        print("接收到数据...............")
        _ = data.withUnsafeBytes {
            self.fileOutputStream?.write($0, maxLength: data.count)
        }
        currentLength += Float(data.count)
        // 这里有个问题 有些写的不规范的 header里面没有length 那就无法计算进度
        let totalLength = Float(dataTask.response?.expectedContentLength ?? -1)
        var p = currentLength / totalLength
        if totalLength<0 {
            p = 0.0
        }
        print("current: \(currentLength) , total:\(totalLength), progress:\(p)")
        self.progress?(p)
    }
    //下载结束 error有值表示失败
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        print("下载完成...........urlSession")
        self.fileOutputStream?.close()
        self.cancel()
        if error != nil {
            self.fail?(String(describing: error))
        }else {
            self.completion?(self.localUrl?.absoluteString ?? "")
        }
    }
}
```

![下载管理.2019-02-22 22_51_44](http://img.muliba.net/%E4%B8%8B%E8%BD%BD%E7%AE%A1%E7%90%86.2019-02-22%2022_51_44.gif)

完整的代码：[SwiftURLSessionDownload](https://github.com/fancylou/SwiftURLSessionDownload)

代码还不完善，只是满足了基本功能，后面会继续完善，并增加断点续传的实现。




