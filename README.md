This is a simple chat demo by using Node.js and Socket.IO.

##基于nodejs服务器，搭建socket.io聊天服务

###从零开始搭建的步骤

- mkdir wqs_node
- cd wqs_node
- npm init
- npm install express --save

```
在index.js写入：
var express = require('express');
var app = express();

app.get('/', function(res, rep) {
    rep.send('Hello, word!');
});

app.listen(3000);
```

- npm index.js 访问localhost就会出现hello world 然后继续下面步骤
- npm install --save socket.io
- 将index.js改写成与本案例index.js


```
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);
var fs = require('fs');

app.get('/', function(req, res){
	res.send('<h1>Welcome Realtime Server</h1>');
});

app.get('/socket', function(req, res){
	fs.readFile('/index.html', 'utf-8',function (err, data) {//读取内容
		if (err) throw err;
		res.writeHead(200, {"Content-Type": "text/html"});//注意这里
		res.write(data);
		res.end();
	});
});

//在线用户
var onlineUsers = {};
//当前在线人数
var onlineCount = 0;

io.on('connection', function(socket){
	console.log('a user connected');
	
	//监听新用户加入
	socket.on('login', function(obj){
		//将新加入用户的唯一标识当作socket的名称，后面退出的时候会用到
		socket.name = obj.userid;
		
		//检查在线列表，如果不在里面就加入
		if(!onlineUsers.hasOwnProperty(obj.userid)) {
			onlineUsers[obj.userid] = obj.username;
			//在线人数+1
			onlineCount++;
		}
		
		//向所有客户端广播用户加入
		io.emit('login', {onlineUsers:onlineUsers, onlineCount:onlineCount, user:obj});
		console.log(obj.username+'加入了聊天室');
	});
	
	//监听用户退出
	socket.on('disconnect', function(){
		//将退出的用户从在线列表中删除
		if(onlineUsers.hasOwnProperty(socket.name)) {
			//退出用户的信息
			var obj = {userid:socket.name, username:onlineUsers[socket.name]};
			
			//删除
			delete onlineUsers[socket.name];
			//在线人数-1
			onlineCount--;
			
			//向所有客户端广播用户退出
			io.emit('logout', {onlineUsers:onlineUsers, onlineCount:onlineCount, user:obj});
			console.log(obj.username+'退出了聊天室');
		}
	});
	
	//监听用户发布聊天内容
	socket.on('message', function(obj){
		//向所有客户端广播发布的消息
		io.emit('message', obj);
		console.log(obj.username+'说：'+obj.content);
	});
  
});

http.listen(3000, function(){
	console.log('listening on *:3000');
});
```

- 打开 index.html 或者 http://localhost:3000/socket页面 输入昵称后  即可使用聊天服务 

## 有帮助的话 点星呀^_^  please click on the star，think you。
