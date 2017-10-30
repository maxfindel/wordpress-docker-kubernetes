# Wordpress-Docker-Kubernetes

Based on 'A slightly less shitty wordpress development workflow'. Please refer to the original Readme file to get all the information: https://github.com/visiblevc/wordpress-starter

## Unencrypt the .env.enc file

TODO.

## Running locally

1. Run the following command in the root of the example directory.
```sh
$ docker-compose up -d && docker-compose logs -f wordpress
```

2. When the build is finished, hit <kbd>ctrl</kbd>-<kbd>c</kbd> to detach from the logs and visit `localhost:8080` in your browser.

## Install the development dependencies

Open the `package.json` file to add your project-specific values. 
You can start with the "name" and "version" values for your project.
In the "gulp" section below, you should add the name of the custom theme you will be developing as well as the AWS variables. It should look like this:

```
{
  "theme": "the-theme",
  "productionAssetURL": "https://s3.amazonaws.com/the-theme-bucket",
  "aws": {
    "params": {
      "Bucket": "the-theme-bucket"
    },
    "accessKeyId": "putyouracceskeyidhere",
    "secretAccessKey": "putyoursecretaccesskeyhere"
  }
}
```

To get started, [install Node](https://nodejs.org/en/download/), [install Gulp](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md) and run `npm install` on the project folder to install the node dependencies.

## Database export / import

If there is a /data/database.sql file in your project the first time your run docker-compose up, it will actually use it to populate the database. It is convenient when you need to setup someone quickly to work on the site.

Here is an example of importing and exporting using the WP CLI:  
```
# restore the database with /data/database.sql
docker exec $(docker ps -lq) /bin/bash -c "wp db import /data/database.sql --allow-root"

# dump the database to /data/database_bk.sql
docker exec $(docker ps -lq) /bin/bash -c "wp db export /data/database_bk.sql --allow-root"
```

## Deploy to Kubernetes

TODO.
https://stackoverflow.com/questions/33478555/kubernetes-equivalent-of-env-file-in-docker

