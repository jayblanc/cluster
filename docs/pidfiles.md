
## PID Files

 Saves out PID files, for example:
 
       master.pid
       worker.0.pid
       worker.1.pid
       worker.2.pid
       worker.3.pid

### Usage

The `pidfiles([path])` plugin saves pid (process-id) files to the given `path` or `./pids`.

save to `./pids`:

     cluster(server)
       .use(cluster.pidfiles())
       .listen(3000);

save to `/var/run/node`:

     cluster(server)
       .use(cluster.logger('/var/run/node'))
       .listen(3000);
