1. MODULE=example.com/foo-controller
2. go mod init $MODULE
3. kubebuilder init --domain example.com
4. kubebuilder edit --multigroup=true
   1. 这一步可以使kubebuilder生成的api目录结构与code-generator保持一致
5. kubebuilder create api --group webapp --version v1 --kind Guestbook
6. 添加文件apis/webapp/v1/rbac.go，这个文件用生成RBAC manifests：

    ```go
   // +kubebuilder:rbac:groups=webapp.example.com,resources=guestbooks,verbs=get;list;watch;create;update;patch;delete
    // +kubebuilder:rbac:groups=webapp.example.com,resources=guestbooks/status,verbs=get;update;patch  

    package v1
   ``` 
   
7. make manifests
8. make install 安装到本地集群
9. code-generator生成client
   ```shell
   .
   └── hack 
      ├── tools.go 
      ├── update-codegen.sh 
      └── verify-codegen.sh 
   # tools.go
   // +build tools 
   package tools 
   import _ "k8s.io/code-generator" 
   # hack/update-codegen.sh
   注意
   MODULE和go.mod保持一致  
   API_PKG=apis，和apis目录保持一致  
   OUTPUT_PKG=generated/webapp，生成Resource时指定的group一样  
   GROUP_VERSION=webapp:v1和生成Resource时指定的group version对应  
   # hack/verify-codegen.sh
   
   # 下载code-generator 注意这里的K8S版本号，得和go.mod里的k8s.io/client-go的版本一致：
   K8S_VERSION=v0.23.0 
   go get k8s.io/code-generator@$K8S_VERSION  
   go mod vendor  
   # 执行
   ./hack/update-codegen.sh
   得到example.com/foo-controller目录：
   example.com/foo-controller/generated直接移出来，放到项目根下面pkg下
   ```

