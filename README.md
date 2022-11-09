一、下载配置文件
  # wget   https://kuboard.cn/install-script/kuboard.yaml

二、修改配置
  # vim kuboard.yaml
    ..........
        spec:
          containers:
          - name: kuboard
            image: eipwork/kuboard:v3
            imagePullPolicy: Always
            volumeMounts:
              - mountPath: /data
                name: data
          volumes:
            - name: data
              hostPath:
                path: /data/kuboard-data
    ..........
# kubectl apply -f kuboard.yaml
三、配置支持 HTTPS
	1、创建 CA 证书
		# openssl req -x509 -sha256 -newkey rsa:4096 -keyout ca.key -out ca.crt -days 356 -nodes -subj '/CN=kuboard.com'
		
	2、创建Server端证书
		# openssl req -new -newkey rsa:4096 -keyout server.key -out server.csr -nodes -subj '/CN=kuboard.com'
	
	3、使用根证书签发Server端请求文件，生成Server端证书
		# openssl x509 -req -sha256 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
	
	4、生成Client端证书的请求文件
		# openssl req -new -newkey rsa:4096 -keyout client.key -out client.csr -nodes -subj '/CN=kuboard.com'
	
	5、使用根证书签发Client端请求文件，生成Client端证书
		# openssl x509 -req -sha256 -days 365 -in client.csr -CA ca.crt -CAkey ca.key -set_serial 02 -out client.crt
	
	6、需改yaml 文件
		# vim kuboard.yaml
                           .......
			        env:
			          - name: KUBOARD_TLS_CERT
			            value: "/etc/certs/kuboard.com/server.crt"
			          - name: KUBOARD_TLS_KEY
			            value: "/etc/certs/kuboard.com/server.key"
			          - name: KUBOARD_ENDPOINT
			            value: "https://kuboard.com"
			          - name: KUBOARD_AGENT_SERVER_UDP_PORT
			            value: "10081"
			          - name: KUBOARD_AGENT_SERVER_TCP_PORT
			            value: "10081"
			        volumeMounts:
			          - mountPath: /data
			            name: data
			          - mountPath: /etc/certs/kuboard.com/server.crt
			            name: server-crt
			          - mountPath: /etc/certs/kuboard.com/server.key
			            name: server-key
			      volumes:
			        - name: data
			          hostPath:
			            path: /data/kuboard-data
			        - name: server-crt
			          hostPath:
			            path: /root/kuboard/tls/server.crt
			        - name: server-key
			          hostPath:
			            path: /root/kuboard/tls/server.key
---
			
	7、访问
            https://kuboard.com:30443/kuboard/cluster
