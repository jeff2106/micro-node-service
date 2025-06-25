# micro-node-service
A basic implementation of a NodeJs Microservice to explain how to use my MicroNodeKit.
 
#What is this?
This is a basic tutorial to explain how to implement a Cluster NodeJs Microservice with a Service Registry 
using a Server Side Discovery pattern.

![ScreenShot](https://raw.github.com/alchimya/micro-node-service/master/MicroservicesArch.png)

![ScreenShot](https://raw.github.com/alchimya/micro-node-service/master/micro-node-net-lib.gif)

#How to install
<b>sudo npm install</b>

#Configuraiton 
The configuraiton is quite straightforward and you must to setup the config.json file as follow:
```javascript
{
  "server":{
    "id":"MicroNodeService",
    "port":8080,
    "isCluster":true,
    "https":{
      "isEnabled":false,
      "key":"",
      "ca":""
    },
    "headers":[
      {"name":"Access-Control-Allow-Origin","value":"*"},
      {"name":"Access-Control-Allow-Headers","value":"Origin, X-Requested-With, Content-Type, Accept"},
      {"name":"Access-Control-Allow-Methods","value":"GET,PUT,POST,DELETE,OPTIONS"}
    ]
  },
  "api":{
    "route":"api",
    "modules":[
      {"name":"login", "route":"login"}
    ]
  },
  "serviceRegistry":{
      "watchDog":{
        "isEnabled":false,
        "timer":30000
      },
      "database":{
        "name":"my_mongo_db",
        "user":"my_mongo_db_user",
        "password":"my_mongo_db_password",
        "host":"my_mongo_db_host",
        "port":27017
      }
  }
}
```
where:
- server.id: it is a key to identify the microservice
- server.port: it is the port where the microservice is listen to
- server,isCluster: set true if you want you fork the main process depending of your CPU
- server.https: enable/disable the https. It is needed to have the righ certificates to allow clients to connect under https
- server.headers: put here all the response headers that you want to use, for example to enable the cross-origin resource sharing.

- api.route: defines the main route of the api (e.g. /api)
- api.modules: add in to this array all the module (js files for example Express middleware) that you want to use as api modules

- serviceRegistry.watchDog: enable/disable the auto-update for the service registry. Speficy the update seconds into the timer property
- serviceRegistry.databse: configure here your MongoDb connection params. This database represents your Service Registry

#How does it work?
The basic idea of this Project is to explain how to implement a basic Login microservice (implementation is not done!) implementing a Service Registry with a Server-Side Discovery pattern. 
<br/>
As you can see from the source code the service can be described as follow:
- process clustering
```javascript
    for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    cluster.on('exit', function(worker, code, signal) {
        console.log("cluster exit");
        debug('Worker %d died with code/signal %s.', worker.process.pid, signal || code);
        debug( Restarting worker...);
        cluster.fork();
    });
```
- express app setup
```javascript
  var app=express();
  app.use(bodyParser.urlencoded({ extended: true }));
  app.use(bodyParser.json());
  app.use(morgan('dev'));
  configServer.express.app = app;
```

- server setup with a service registry on the success callback
```javascript
    server.create(function (err,server) {
        if (!err){
            registerServer();
        }
    }); 
```

- server exit handler registration that allow to unregister the service
```javascript
  server.registerExitHandler(function () {
      unregisterServer();
  });
```
- express error andling

```javascript
  app.use(function(err, req, res, next) {
      //LOG HERE ERROR WITH A LOG ERRORS MICROSERVICE ;-)
      debug(err);
      res.status(500).send({
          code:1000,
          message:"Application Error",
          domain:"Application Error"
      });
  });
```
Enjopy ;-)





