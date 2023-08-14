### I. Init project

```
nvm use v18.16.1
mkdir cluster_demo
cd cluster_demo
npm init -y
npm install express
npm install -g loadtest pm2
```

#### MindNode

![clustering mindnode](/images/mindnode.png "Clustering Mindnode")

### II. Single thread

Since you have a single process for the Node.js application running on a server with multiple CPUs, it will receive and handle all incoming requests. In this diagram, all the incoming requests are directed to the process running on a single CPU while the other CPUs stay idle:

![single-thread](/images/single-thread.png "Single Thread Diagram")

Now that you have created an app without using the cluster module, you will use the cluster module to scale the application to use multiple CPUs next.

Checking `index.js`

```
node index.js

worker pid=2600
App listening on port 3000
```

### III. Clustering

In this step, you will add the cluster module to create multiple instances of the same program to handle more load and improve performance. When you run processes with the cluster module, you can have multiple processes running on each CPU on your machine:

![clustering](/images/clustering.png "Clustering Diagram")

Checking `primary.js`

```
node primary.js

The total number of CPUs is 4
Primary pid=2484
worker pid=2485
App listening on port 3000
worker pid=2488
worker pid=2486
App listening on port 3000
worker pid=2487
App listening on port 3000
App listening on port 3000

```

### IV. Comparing Performance using a Load Testing Tool

#### Testing with Single Thread

In this step, you will use the loadtest package to generate traffic against the two programs you’ve built. You’ll compare the performance of the `primary.js` program which uses the cluster module with that of the `index.js` program which does not use clustering. You will notice that the program using the cluster module performs faster and can handle more requests within a specific time than the program that doesn’t use clustering.

1. In your first terminal, run the index.js file to start the server:

2. Running test: `loadtest -n 1200 -c 200 -k http://localhost:3000/heavy`

3. See output:

```
[Mon Aug 14 2023 15:30:41 GMT+0700 (Indochina Time)] INFO Requests: 0 (0%), requests per second: 0, mean latency: 0 ms
[Mon Aug 14 2023 15:30:46 GMT+0700 (Indochina Time)] INFO Requests: 562 (47%), requests per second: 113, mean latency: 1399.3 ms
[Mon Aug 14 2023 15:30:51 GMT+0700 (Indochina Time)] INFO Requests: 1111 (93%), requests per second: 110, mean latency: 1899.3 ms
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Target URL:          http://localhost:3000/heavy
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Max requests:        1200
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Concurrency level:   200
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Agent:               keepalive
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Completed requests:  1200
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Total errors:        0
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Total time:          10.845923209 s
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Requests per second: 111
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Mean latency:        1653.2 ms
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO Percentage of the requests served within a certain time
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO   50%      1737 ms
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO   90%      2142 ms
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO   95%      3151 ms
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO   99%      3297 ms
[Mon Aug 14 2023 15:30:52 GMT+0700 (Indochina Time)] INFO  100%      3327 ms (longest request)
```

- Total time measures how long it took for all the requests to be served. In this output, it took just over 10 seconds to serve all 1200 requests.

- Requests per second measures the number of requests the server can handle per second. In this output, the server handles 111 requests per second.

- Mean latency measures the time it took to send a request and get a response, which is 1653.2 ms in the sample output.

#### Testing with Clustering

See output:

```
[Mon Aug 14 2023 15:33:53 GMT+0700 (Indochina Time)] INFO Requests: 0 (0%), requests per second: 0, mean latency: 0 ms
[Mon Aug 14 2023 15:33:58 GMT+0700 (Indochina Time)] INFO Requests: 1018 (85%), requests per second: 204, mean latency: 467 ms
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Target URL:          http://localhost:3000/heavy
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Max requests:        1200
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Concurrency level:   200
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Agent:               keepalive
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Completed requests:  1200
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Total errors:        0
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Total time:          5.953313419 s
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Requests per second: 202
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Mean latency:        900.9 ms
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO Percentage of the requests served within a certain time
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO   50%      314 ms
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO   90%      4684 ms
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO   95%      4772 ms
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO   99%      4804 ms
[Mon Aug 14 2023 15:33:59 GMT+0700 (Indochina Time)] INFO  100%      4837 ms (longest request)
```

- Total time measures how long it took for all the requests to be served. In this output, it took just over 5 seconds to serve all 1200 requests.

- Requests per second measures the number of requests the server can handle per second. In this output, the server handles 202 requests per second.

- Mean latency measures the time it took to send a request and get a response, which is 900.9 ms in the sample output.

#### Using PM2 for Clustering

So far, you have used the cluster module to create worker processes according to the number of CPUs on your machine. You have also added the ability to restart a worker process when it dies. In this step, you will set up an alternative to automate scaling your app by using the pm2 process manager, which is built upon the cluster module. This process manager contains a load balancer and can automatically create as many worker processes as CPUs on your machine. It also allows you to monitor the processes and can spawn a new worker process automatically if one dies.

In your initial terminal, start the pm2 cluster with the following command:

`pm2 start index.js -i 0`

The -i option accepts the number of worker processes

 > If you pass the argument 0, pm2 will automatically create as many worker processes as there are CPUs on your machine.

```
 pm2 start index.js -i 0
[PM2] Starting /Users/kietnvt/workspace/nodejs_clustering_demo/index.js in cluster_mode (0 instance)
[PM2] Done.
┌────┬──────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name     │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├────┼──────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0  │ index    │ default     │ 1.0.0   │ cluster │ 5673     │ 0s     │ 0    │ online    │ 19%      │ 47.4mb   │ kietnvt  │ disabled │
│ 1  │ index    │ default     │ 1.0.0   │ cluster │ 5674     │ 0s     │ 0    │ online    │ 0%       │ 45.3mb   │ kietnvt  │ disabled │
│ 2  │ index    │ default     │ 1.0.0   │ cluster │ 5675     │ 0s     │ 0    │ online    │ 0%       │ 31.3mb   │ kietnvt  │ disabled │
│ 3  │ index    │ default     │ 1.0.0   │ cluster │ 5676     │ 0s     │ 0    │ online    │ 0%       │ 37.3mb   │ kietnvt  │ disabled │
└────┴──────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
[PM2][WARN] Current process list is not synchronized with saved list. App test differs. Type 'pm2 save' to synchronize.
```

Reading logs:

```
0|index    | Server: Listening on port 8080 in development mode
0|index    | Server: App has admin account already !
0|index    | ::1 - - [10/Mar/2023:06:57:59 +0000] "GET / HTTP/1.1" 200 35 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36"
0|index    | ::1 - - [10/Mar/2023:06:58:00 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36"
0|index    | worker pid=5673
0|index    | App listening on port 3000
0|index    | worker pid=5674
0|index    | App listening on port 3000
0|index    | worker pid=5675
0|index    | App listening on port 3000
0|index    | worker pid=5676
0|index    | App listening on port 3000
```

Now we can load test with `loadtest -n 1200 -c 200 -k http://localhost:3000/heavy`

```
[Mon Aug 14 2023 15:40:20 GMT+0700 (Indochina Time)] INFO Requests: 0 (0%), requests per second: 0, mean latency: 0 ms
[Mon Aug 14 2023 15:40:25 GMT+0700 (Indochina Time)] INFO Requests: 960 (80%), requests per second: 193, mean latency: 482.1 ms
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Target URL:          http://localhost:3000/heavy
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Max requests:        1200
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Concurrency level:   200
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Agent:               keepalive
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Completed requests:  1200
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Total errors:        1
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Total time:          6.3305173770000005 s
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Requests per second: 190
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Mean latency:        961.1 ms
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO Percentage of the requests served within a certain time
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO   50%      361 ms
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO   90%      5006 ms
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO   95%      5121 ms
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO   99%      5144 ms
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO  100%      5194 ms (longest request)
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO
[Mon Aug 14 2023 15:40:27 GMT+0700 (Indochina Time)] INFO  100%      5194 ms (longest request)
```

Next, generate a configuration file:

`pm2 ecosystem`

Rename .js to .cjs to enable support for ES modules

`mv ecosystem.config.js ecosystem.config.cjs`

In your ecosystem.config.cjs file, add the highlighted code below:

```
module.exports = {
  apps : [{
    script: 'index.js',
    watch: '.',
    name: "cluster_app",
    instances: 0,
    exec_mode: "cluster",
  }, {
    script: './service-worker/',
    watch: ['./service-worker']
  }],

  deploy : {
    production : {
      user : 'SSH_USERNAME',
      host : 'SSH_HOSTMACHINE',
      ref  : 'origin/master',
      repo : 'GIT_REPOSITORY',
      path : 'DESTINATION_PATH',
      'pre-deploy-local': '',
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.cjs --env production',
      'pre-setup': ''
    }
  }
};
```

Next, start with ecosystem: `pm2 start ecosystem.config.cjs`

You will have clustering running:

```
┌────┬────────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name           │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├────┼────────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0  │ cluster_app    │ default     │ 1.0.0   │ cluster │ 6240     │ 4s     │ 0    │ online    │ 0%       │ 56.0mb   │ kietnvt  │ enabled  │
│ 1  │ cluster_app    │ default     │ 1.0.0   │ cluster │ 6241     │ 3s     │ 0    │ online    │ 0%       │ 55.3mb   │ kietnvt  │ enabled  │
│ 2  │ cluster_app    │ default     │ 1.0.0   │ cluster │ 6242     │ 3s     │ 0    │ online    │ 0%       │ 54.2mb   │ kietnvt  │ enabled  │
│ 3  │ cluster_app    │ default     │ 1.0.0   │ cluster │ 6243     │ 3s     │ 0    │ online    │ 0%       │ 55.9mb   │ kietnvt  │ enabled  │
└────┴────────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
````

Reference:

https://www.digitalocean.com/community/tutorials/how-to-scale-node-js-applications-with-clustering

Next:

https://www.digitalocean.com/community/tutorials/how-to-handle-asynchronous-tasks-with-node-js-and-bullmq