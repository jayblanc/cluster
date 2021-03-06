
## API

 The Cluster API at its core is extremely simple, all we need to do is pass
 our http `server` to `cluster()`, then call `listen()` as we would on the `http.Server` itself.


     var cluster = require('../')
       , http = require('http');

     var server = http.createServer(function(req, res){
       res.writeHead(200);
       res.end('Hello World');
     });

     cluster(server)
       .listen(3000);

### Plugins

 A plugin simple a function that accepts the `master` process. Most plugin functions _return_ another anonymous function, allowing them to accept options, for example:
 
    function myPlugin(path){
      return function(master) {
        // do stuff
      }
    }

 To use them, all we need to do is pass it to the `use()` method:
 
    cluster(server)
      .use(myPlugin('/some/path'))
      .listen(3000);

 To use a plugin that is bundled with Cluster simply grab it from the `cluster` object:
 
     cluster(server)
       .use(cluster.logger())
       .listen(3000);

### Settings

 Below are the settings available:
 
   - `workers`  Number of workers to spawn, defaults to the number of CPUs or `1`
   - `working directory`  Working directory defaulting to `/`
   - `backlog`  Connection backlog, defaulting to 128
   - `socket path`  Master socket path defaulting to `./`
   - `timeout` Worker shutdown timeout in milliseconds, defaulting to `60000`
   - `user`  User id / name
   - `group`  Group id / name

 We can take what we have now, and go on to apply settings using the `set(option, value)` method. For example:
 
    cluster(server)
      .set('working directory', '/')
      .set('workers', 5)
      .listen(3000);

### Signals

 Cluster performs the following actions when handling signals:
 
   - `SIGINT`   hard shutdown
   - `SIGTERM`  hard shutdown
   - `SIGQUIT`  graceful shutdown
   - `SIGUSR2`  restart workers
   - `SIGHUP`   restart workers

### Events

 The following events are emitted, useful for plugins or general purpose logging etc.
 
   - `start`. When the server is starting (pre-spawn)
   - `worker`. When a worker is spawned, passing the `worker`
   - `listening`. When the server is listening for connections (post-spawn)
   - `closing`. When master is gracefully shutting down
   - `close`. When master has completed shutting down
   - `worker killed`. When a worker has died
   - `kill`. When a `signal` is being sent to all workers
   - `restarting`. Restart requested by REPL or signal. Receives an object
     which can be patched in order to preserve plugin state.
   - `restart`. Restart complete, new master established, previous killed.
     Receives an object with state preserved by the `restarting` even,
     patched in the previous master.

### Master#state

 Current state of the master process, one of:
 
   - `active`
   - `hard shutdown`
   - `graceful shutdown`

### Master#isWorker

 `true` when the script is executed as a worker.

      cluster = cluster(server).listen(3000);

      if (cluster.isWorker) {
        // do something
      }

### Master#isMaster

`true` when the script is executed as master.

     cluster = cluster(server).listen(3000);

     if (cluster.isMaster) {
       // do something
     }

### Master#set(option, value)

  Set `option` to `value`.

### Master#use(plugin)

  Register a `plugin` for use.

### Master#in(env)

 Conditionally perform the following action, if 
 __NODE_ENV__ matches `env`.

     cluster(server)
       .in('development').use(cluster.debug())
       .in('development').listen(3000)
       .in('production').listen(80);

 The environment conditionals may be applied to several calls:
 
     cluster(server)
       .set('working directory', '/')
       .in('development')
         .set('workers', 1)
         .use(cluster.logger('logs', 'debug'))
         .use(cluster.debug())
         .listen(3000)
       .in('production')
         .set('workers', 4)
         .use(cluster.logger())
         .use(cluster.pidfiles())
         .listen(80);
 

### Master#spawn(n)

  Spawn `n` additional workers.

### Master#close()

  Graceful shutdown, waits for all workers to reply before exiting.

### Master#destroy()

  Hard shutdown, immediately kill all workers.

### Master#restart([signal])

  Defaults to a graceful restart, spawning a new master process, and sending __SIGQUIT__ to the previous master process. Alternatively a custom `signal` may be passed.

### Master#kill([signal])

 Sends __SIGTERM__ or `signal` to all worker processes. This method is used by `Master#restart()`, `Master#close()` etc.