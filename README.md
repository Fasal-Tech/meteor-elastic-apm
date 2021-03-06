# meteor-elastic-apm

Performance Monitoring for Meteor based on Elastic APM

[![Meteor Elastic APM screenshot](https://github.com/kschingiz/meteor-elastic-apm/blob/master/assets/meteor-transaction.png)](https://github.com/kschingiz/meteor-elastic-apm)

# what it monitors

1. Meteor methods - detailed stack trace of what happened insine methods
2. Meteor pub/sub - tracks publications response time
3. Meteor pub/sub - operations, how many documents were added, updated, removed
4. Async ops inside methods and pubs, for example http requests
5. All Incoming and outcoming TCP requests, usefull if you have REST API
6. Exception handling

# more screenshots

https://github.com/kschingiz/meteor-elastic-apm/blob/master/assets/

# project status - Production Ready

# getting started

1. Install and configure elasticsearch - https://www.elastic.co/downloads/elasticsearch
2. Install and configure Kibana - https://www.elastic.co/downloads/kibana
3. Install and configure elastic APM server - https://www.elastic.co/downloads/apm

Then

```bash
meteor add kschingiz:meteor-elastic-apm
```

Maybe you will need to also install

```bash
meteor add http mongo-livequery
```

Then somewhere in your server code, Elastic documentation stays that Agent.start should be executed before anything else, and should be at the very top of your code

```js
import Agent from "meteor/kschingiz:meteor-elastic-apm";

const options = {
  serviceName: "meteor-demo-app"
};
Agent.start(options);
```

Complete [list of Agent options](https://www.elastic.co/guide/en/apm/agent/nodejs/current/advanced-setup.html)

# Kibana APM with Meteor with MUP

Meteor Up is a production quality Meteor app deployment tool. We expect you already has up and running Meteor app on server deployed with MUP.

1. `mup ssh`
2. `wget https://raw.githubusercontent.com/elastic/apm-server/master/apm-server.yml && cp apm-server.yml /etc/apm-server/apm-server.yml`
3. Now you need to edit /etc/apm-server/apm-server.yml, at least you need to add you elastic search url under `output.elasticsearch`. When you finish just close this terminal
4. Now we need to update mup.js file to:
   a) Install apm-server in app container
   b) Pass apm-server config file into our app container
   c) Start it everytime after deploy

```
{
  app: {
    ...
    volumes: {
      '/etc/apm-server/apm-server.yml': '/etc/apm-server/apm-server.yml'
    },
    docker: {
        ...
        buildInstructions: [
        // https://www.elastic.co/guide/en/apm/server/current/setup-repositories.html
        'RUN apt-get install wget -y',
        'RUN wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -',
        'RUN apt-get install apt-transport-https',
        'RUN echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-6.x.list',
        'RUN apt-get update && apt-get install apm-server -y',
        'RUN update-rc.d apm-server defaults 95 10'
        ]
    }
    ...
  },
  ...
  hooks: {
    // Run apm-server
    'post.deploy'(api) {
      return api.runSSHCommand(
        api.getConfig().servers.one,
        'docker exec development service apm-server start'
      );
    }
  },
}
```

# demo app

https://github.com/kschingiz/demo-meteor-elastic-apm

# API

Agent is based on `elastic/apm-agent-nodejs` and fully supports all of it's features https://github.com/elastic/apm-agent-nodejs

# contributions

All contributions are welcome, Let's make the better APM together!
