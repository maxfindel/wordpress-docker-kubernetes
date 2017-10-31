# Wordpress-Docker-Kubernetes

Based on 'A slightly less shitty wordpress development workflow'. 

It is highly recommended that you read or skim the original articles here:  
https://visible.vc/engineering/docker-environment-for-wordpress/

Please refer to the original README file to get all the information:  
https://github.com/visiblevc/wordpress-starter

## Getting started

In this repo you will find a `docker-compose.yml` file as well as a few configuration files that will help you improve your development workflow and will help you automate your deployment processes.

TODO Unencrypt the .env.enc file.

## Running locally

1. Run the following command in the root of the example directory.
```sh
$ docker-compose up -d && docker-compose logs -f wordpress
```

2. When the build is finished, hit <kbd>ctrl</kbd>-<kbd>c</kbd> to detach from the logs and visit `localhost:8080` in your browser.

## Development dependencies

You need an AWS User and a Bucket for your assets. It is recommended to create a special set of KEY/SECRET that only has access to this bucket. For this you need to create a new User (I call it just like the bucket) in the AIM section of AWS and attach a custom AIM policy. The custom AIM policy should look like this, replacing 'the-theme-bucket' with the name of your bucket:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:ListAllMyBuckets",
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::the-theme-bucket"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::the-theme-bucket/*"
            ]
        }
    ]
}
```

TODO populate the package.json using the .env file.

Open the `package.json` file to add your project-specific values. 
You can start with the "name" and "version" values for your project.  

In the "gulp" section below, you should add the name of the "custom theme" you will be developing as well as the AWS variables. 

The `package.json` should look like this, replacing your values for 'the-theme' and AWS:

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

If you are developing a new Theme, you should add it to the `.gitignore` file as well.

## Database export / import

If there is a /data/database.sql file in your project the first time your run docker-compose up, it will actually use it to populate the database. It is convenient when you need to setup someone quickly to work on the site.

Here is an example of importing and exporting using the WP CLI:  
```
# restore the database with /data/database.sql
docker exec $(docker ps -lq) /bin/bash -c "wp db import /data/database.sql --allow-root"

# dump the database to /data/database_bk.sql
docker exec $(docker ps -lq) /bin/bash -c "wp db export /data/database_bk.sql --allow-root"
```

## Development

To monitor your assets changes and automatically add them to your `dist` folder, you want to have an open terminal running the command `gulp` while you develop.

Now you are ready to start developing your theme or managing your website using WordPress!

## First deploy to Kubernetes

TODO create a Rancher Cluster.

TODO Unencrypt the certs folder and replace your credentials.

TODO Upload your services and deployments.

## Continuous deployment in Kubernetes

To make a deployment, the files in the `dist` folder need to be processed for production and exported to Amazon S3. You can do this with the command `gulp deploy-assets`. 

If you use an automated tool to build and deploy your projects, all the necessary information is in the .env file and all you need to have is the encryption-password.

In this repo you can find a bitbucket-pipelines.yml file where all the commands necessary to build this images are run and then it gets deployed to Kubernetes.

TODO add bitbucket-pipelines.

