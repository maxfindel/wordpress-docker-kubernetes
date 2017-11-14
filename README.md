# Wordpress-Docker-Kubernetes

Based on 'A slightly less shitty wordpress development workflow'. 

It is highly recommended that you read or skim the original articles here:  
https://visible.vc/engineering/docker-environment-for-wordpress/

Please refer to the original README file to get all the information:  
https://github.com/visiblevc/wordpress-starter

## Getting started

In this repo you will find a `docker-compose.yml` file as well as a few configuration files that will help you improve your development workflow and will help you automate your deployment processes.

First, we want to unencrypt the .env.enc file. In this encrypted file we will store all the confidential keys and the configuration for our project. This way, we can deploy these variables to the repo and make them easily available for CI Tools and other team members.

Unencrypt the original .env.enc with the following command and obtain a .env file.
```
openssl enc -aes-256-cbc -salt -d -in .env.enc -out .env -k secretpassword
```

This file contains a 'docker-compose config' section that can be used directly in the docker-compose file. Just run the command `docker-compose config` to check the output.

The second section of this file is the 'package.json config'. Here you can find your project-specific values as well as your AWS credentials (a sample) and bucket. Once you have your own values added to this file, you can replace them in the (ignored) package.json file using:
```
source .env
sed "s;<PROJECT_NAME>;$PROJECT_NAME;g" package.json.tmp > package.json.tmp1
sed "s;<PROJECT_VERSION>;$PROJECT_VERSION;g" package.json.tmp1 > package.json.tmp2
sed "s;<THEME_NAME>;$THEME_NAME;g" package.json.tmp2 > package.json.tmp1
sed "s;<AWS_BUCKET>;$AWS_BUCKET;g" package.json.tmp1 > package.json.tmp2
sed "s;<AWS_KEY>;$AWS_KEY;g" package.json.tmp2 > package.json.tmp1
sed "s;<AWS_SECRET>;$AWS_SECRET;g" package.json.tmp1 > package.json
rm package.json.tmp1
rm package.json.tmp2
```

To store changes made to the .env file and commit them (as the .env file is ignored), you want to re-encrypt the file using your own SECRET PASSOWRD (that you don't post anywhere).
```
openssl enc -aes-256-cbc -salt -in .env -out .env.enc -k areallysecretpassword
```

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

To populate the package.json file using the .env file, just run the command listed in the previous section.

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

To stop the containers or restart them just tun `docker-compose stop`.

## First deploy to Kubernetes

TODO create a Rancher Cluster.

TODO Unencrypt the certs folder and replace your credentials.

TODO Upload your services and deployments.

## Continuous deployment in Kubernetes

To make a deployment, the files in the `dist` folder need to be processed for production and exported to Amazon S3. You can do this with the command `gulp deploy-assets`. 

If you use an automated tool to build and deploy your projects, all the necessary information is in the .env file and all you need to have is the encryption-password.

In this repo you can find a bitbucket-pipelines.yml file where all the commands necessary to build this images are run and then it gets deployed to Kubernetes.

TODO add bitbucket-pipelines.

