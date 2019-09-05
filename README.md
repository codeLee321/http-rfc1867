# RFC1867（文件上传） 协议分析

### 1.用例:
```html
<FORM ACTION="http://server.dom/cgi/handle"
           ENCTYPE="multipart/form-data"
           METHOD=POST>
    What is your name? <INPUT TYPE=TEXT NAME=submitter>
    What files are you sending? <INPUT TYPE=FILE NAME=pics>
</FORM>
```
在使用表单提交文件的时候，整个表单数据在组装和输出的过程完全是由``FORM``表单来完成的，那么它是按照什么样的规则来组装这些数据的呢？这就是我本次要分析的文件传输使用的[RFC1867](https://www.ietf.org/rfc/rfc1867.txt)协议。

### 2.传输数据展示
```
#请求头设置
Content-type: multipart/form-data, boundary=AaB03x                #设置请求头部类型

#请求体设置
--AaB03x                                                          #表单中某个数据的开始，下一部分换行（\r\n）
content-disposition: form-data; name="submitter"                  #表单中参数的Key，下一部分换行（\r\n\r\n）

Joe Blow                                                          #表单中参数的Value，下一部分换行（\r\n）
--AaB03x                                                          #表单中下个数据的开始，下一部分换行（\r\n）
content-disposition: form-data; name="pics"; filename="file1.txt" #表单中文件的Key和文件名称（\r\n）
Content-Type: text/plain                                          #表单中文件类型（\r\n\r\n）

... contents of file1.txt ...                                     #表单中文件的实体部分（\r\n）
--AaB03x--                                                        #表单数据传输结束标示（\r\n）
```
注意：上面传输数据包含来两部分：普通参数和文本类型；普通参数的key-value之间是由``\r\n\r\n``隔开，而文本类型中，文件的类型和实体之间是由``\r\n\r\n``隔开，其他则由``\r\n``相隔。切记不要遗漏请求体内容的结束标示``--boundary-- ``

### 3.抓包数据展示
```
#请求头数据
multipart/form-data; boundary=----WebKitFormBoundaryT6Bt8Wd1IDDQYKBW

#请求头体数据
------WebKitFormBoundaryIAle5fTnX18fxZYz
Content-Disposition: form-data; name="submitter"

codelee
------WebKitFormBoundaryIAle5fTnX18fxZYz
Content-Disposition: form-data; name="pics"; filename="828.jpg"
Content-Type: image/jpeg

ff e1 09 50 68 74 74 70 3a 2f 2f 6e 73 2e 61 64      Phttp://ns.ad
6f 62 65 2e 63 6f 6d 2f 78 61 70 2f 31 2e 30 2f   obe.com/xap/1.0/
00 3c 3f 78 70 61 63 6b 65 74 20 62 65 67 69 6e    <?xpacket begin
3d 22 ef bb bf 22 20 69 64 3d 22 57 35 4d 30 4d   ="   " id="W5M0M
70 43 65 68 69 48 7a 72 65 53 7a 4e 54 63 7a 6b   pCehiHzreSzNTczk
63 39 64 22 3f 3e 20 3c 78 3a 78 6d 70 6d 65 74   c9d"?> <x:xmpmet
61 20 78 6d 6c 6e 73 3a 78 3d 22 61 64 6f 62 65   a xmlns:x="adobe
3a 6e 73 3a 6d 65 74 61 2f 22 20 78 3a 78 6d 70   :ns:meta/" x:xmp
74 6b 3d 22 41 64 6f 62 65 20 58 4d 50 20 43 6f   tk="Adobe XMP Co
72 65 20 35 2e 36 2d 63 31 34 30 20 37 39 2e 31   re 5.6-c140 79.1
36 30 34 35 31 2c 20 32 30 31 37 2f 30 35 2f 30   60451, 2017/05/0
36 2d 30 31 3a 30 38 3a 32 31 20 20 20 20 20 20   6-01:08:21      
20 20 22 3e 20 3c 72 64 66 3a 52 44 46 20 78 6d     "> <rdf:RDF xm
6c 6e 73 3a 72 64 66 3d 22 68 74 74 70 3a 2f 2f   lns:rdf="http://
77 77 77 2e 77 33 2e 6f 72 67 2f 31 39 39 39 2f   www.w3.org/1999/
30 32 2f 32 32 2d 72 64 66 2d 73 79 6e 74 61 78   02/22-rdf-syntax
2d 6e 73 23 22 3e 20 3c 72 64 66 3a 44 65 73 63   -ns#"> <rdf:Desc
72 69 70 74 69 6f 6e 20 72 64 66 3a 61 62 6f 75   ription rdf:abou
74 3d 22 22 2f 3e 20 3c 2f 72 64 66 3a 52 44 46   t=""/> </rdf:RDF
3e 20 3c 2f 78 3a 78 6d 70 6d 65 74 61 3e 20 20   > </x:xmpmeta>  
20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20                   
20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 20 
...
------WebKitFormBoundaryIAle5fTnX18fxZYz--
```
注意：上面是浏览器的抓包数据，这里它放了一个烟雾弹，boundary的实际值为``----WebKitFormBoundaryT6Bt8Wd1IDDQYKBW``，所以在下面数据开始的标示有六个``-``，误让你以为六个``-``开始，实际上里面的四个``-``是boundary的值。

### 4.安卓&iOS没有这种表单组件改如何上传文本内容的呢？原理类似，遵循RFC1867协议规范：
ios如下，安卓雷同
```swift
import UIKit
import MobileCoreServices
 
class ViewController: UIViewController {
     
    @IBAction func startUpload(_ sender: Any) {
         
        //分隔线
        let boundary = "Boundary-\(UUID().uuidString)"
         
        //传递的参数
        let parameters = [
            "value1": "hangge.com",
            "value2": "1234"
        ]
         
        //传递的文件
        let files = [
            (
                name: "file1",
                path:Bundle.main.path(forResource: "1", ofType: "jpg")!
            ),
            (
                name: "file2",
                path:Bundle.main.path(forResource: "2", ofType: "png")!
            )
        ]
         
        //上传地址
        let url = URL(string: "http://www.hangge.com/upload.php")!
        var request = URLRequest(url: url)
        //请求类型为POST
        request.httpMethod = "POST"
        request.setValue("multipart/form-data; boundary=\(boundary)",
            forHTTPHeaderField: "Content-Type")
         
        //创建表单body
        request.httpBody = try! createBody(with: parameters, files: files, boundary: boundary)
         
        //创建一个表单上传任务
        let session = URLSession(configuration: .default, delegate: self, delegateQueue: nil)
        let uploadTask = session.dataTask(with: request, completionHandler: {
            (data, response, error) -> Void in
            //上传完毕后
            if error != nil{
                print(error!)
            }else{
                let str = String(data: data!, encoding: String.Encoding.utf8)
                print("--- 上传完毕 ---\(str!)")
            }
        })
         
        //使用resume方法启动任务
        uploadTask.resume()
    }
     
    //创建表单body
    private func createBody(with parameters: [String: String]?,
                            files: [(name:String, path:String)],
                            boundary: String) throws -> Data {
        var body = Data()
         
        //添加普通参数数据
        if parameters != nil {
            for (key, value) in parameters! {
                // 数据之前要用 --分隔线 来隔开 ，否则后台会解析失败
                body.append("--\(boundary)\r\n")
                body.append("Content-Disposition: form-data; name=\"\(key)\"\r\n\r\n")
                body.append("\(value)\r\n")
            }
        }
         
        //添加文件数据
        for file in files {
            let url = URL(fileURLWithPath: file.path)
            let filename = url.lastPathComponent
            let data = try Data(contentsOf: url)
            let mimetype = mimeType(pathExtension: url.pathExtension)
             
            // 数据之前要用 --分隔线 来隔开 ，否则后台会解析失败
            body.append("--\(boundary)\r\n")
            body.append("Content-Disposition: form-data; "
                + "name=\"\(file.name)\"; filename=\"\(filename)\"\r\n")
            body.append("Content-Type: \(mimetype)\r\n\r\n") //文件类型
            body.append(data) //文件主体
            body.append("\r\n") //使用\r\n来表示这个这个值的结束符
        }
         
        // --分隔线-- 为整个表单的结束符
        body.append("--\(boundary)--\r\n")
        return body
    }
     
    //根据后缀获取对应的Mime-Type
    func mimeType(pathExtension: String) -> String {
        if let uti = UTTypeCreatePreferredIdentifierForTag(kUTTagClassFilenameExtension,
                                                           pathExtension as NSString,
                                                           nil)?.takeRetainedValue() {
            if let mimetype = UTTypeCopyPreferredTagWithClass(uti, kUTTagClassMIMEType)?
                .takeRetainedValue() {
                return mimetype as String
            }
        }
        //文件资源类型如果不知道，传万能类型application/octet-stream，服务器会自动解析文件类
        return "application/octet-stream"
    }
}
 
extension ViewController: URLSessionDelegate, URLSessionTaskDelegate {
    //上传代理方法，监听上传进度
    func urlSession(_ session: URLSession, task: URLSessionTask,
                    didSendBodyData bytesSent: Int64, totalBytesSent: Int64,
                    totalBytesExpectedToSend: Int64) {
        //获取进度
        let written = (Float)(totalBytesSent)
        let total = (Float)(totalBytesExpectedToSend)
        let pro = written/total
        print("当前进度：\(pro)")
    }
}
 
//扩展Data
extension Data {
    //增加直接添加String数据的方法
    mutating func append(_ string: String, using encoding: String.Encoding = .utf8) {
        if let data = string.data(using: encoding) {
            append(data)
        }
    }
}

```
