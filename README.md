## Introduction
This is a Dockerfile to build a container image for nginx and php-fpm, with the ability to pull website code from git. The container also has the ability to update templated files with variables passed to docker in order to update your settings. There is also support for lets encrypt SSL support.

### Git repository
The source files for this project can be found here: [https://github.com/ngineered/nginx-php-fpm](https://github.com/ngineered/nginx-php-fpm)

If you have any improvements please submit a pull request.
### Docker hub repository
The Docker hub build can be found here: [https://registry.hub.docker.com/u/richarvey/nginx-php-fpm/](https://registry.hub.docker.com/u/richarvey/nginx-php-fpm/)
## Versions
| Tag | Nginx | PHP | Alpine |
|-----|-------|-----|--------|
| latest | 1.10.1 | 7.0.8 | 3.4 |
| php5 | 1.10.1 | 5.6.23 | 3.4 |
| php7 | 1.10.1 | 7.0.8 | 3.4 |

## Building from source
To build from source you need to clone the git repo and run docker build:
```
git clone https://github.com/ngineered/nginx-php-fpm
.git
docker build -t nginx-php-fpm:latest .
```

## Pulling from Docker Hub
```
docker pull richarvey/nginx-php-fpm
```

## Running
To simply run the container:
```
sudo docker run -d richarvey/nginx-php-fpm
```

You can then browse to ```http://<DOCKER_HOST>:8080``` to view the default install files. To find your ```DOCKER_HOST``` use the ```docker inspect``` to get the IP address.

### Available Configuration Parameters
The following flags are a list of all the currently supported options that can be changed by passing in the variables to docker with the -e flag.

 - **GIT_REPO** : URL to the repository containing your source code. If you are using a personal token, this is the https URL without https://, e.g github.com/project/ for ssh prepend with git@ e.g git@github.com:project.git
 - **GIT_BRANCH** : Select a specific branch (optional)
 - **GIT_EMAIL** : Set your email for code pushing (required for git to work)
 - **GIT_NAME** : Set your name for code pushing (required for git to work)
 - **SSH_KEY** : Private SSH deploy key for your repository base64 encoded (requires write permissions for pushing)
 - **GIT_PERSONAL_TOKEN** : Personal access token for your git account (required for HTTPS git access)
 - **GIT_USERNAME** : Git username for use with personal tokens. (required for HTTPS git access)
 - **WEBROOT** : Change the default webroot directory from `/var/www/html` to your own setting
 - **ERRORS** : Set to 1 to display PHP Errors in the browser
 - **HIDE_NGINX_HEADERS** : Disable by setting to 0, default behaviour is to hide nginx + php version in headers
 - **PHP_MEM_LIMIT** : Set higher PHP memory limit, default is 128 Mb
 - **PHP_POST_MAX_SIZE** : Set a larger post_max_size, default is 100 Mb
 - **PHP_UPLOAD_MAX_FILESIZE** : Set a larger upload_max_filesize, default is 100 Mb
 - **DOMAIN** : Set domain name for Lets Encrypt scripts
 - **RUN_SCRIPTS** : Set to 1 to execute scripts

### Dynamically Pulling code from git
One of the nice features of this container is its ability to pull code from a git repository with a couple of environmental variables passed at run time. Please take a look at our recommended [repo layout guidelines](https://github.com/ngineered/nginx-php-fpm/blob/master/docs/repo_layout.md).

There are two methods of pulling code from git, you can either use a Personal Token (recommended method) or an SSH key.

**Note:** We would recommend using a git personal token over an SSH key as it simplifies the set up process. To create a personal access token on Github follow this [guide](https://help.github.com/articles/creating-an-access-token-for-command-line-use/).

#### Personal Access token

You can pass the container your personal access token from your git account using the __GIT_PERSONAL_TOKEN__ flag. This token must be setup with the correct permissions in git in order to push and pull code.

Since the access token acts as a password with limited access, the git push/pull uses HTTPS to authenticate. You will need to specify your __GIT_USERNAME__ and __GIT_PERSONAL_TOKEN__ variables to push and pull. You'll need to also have the __GIT_EMAIL__, __GIT_NAME__ and __GIT_REPO__ common variables defined.

```
docker run -d -e 'GIT_EMAIL=email_address' -e 'GIT_NAME=full_name' -e 'GIT_USERNAME=git_username' -e 'GIT_REPO=github.com/project' -e 'GIT_PERSONAL_TOKEN=<long_token_string_here>' richarvey/nginx-php-fpm:latest
```

To pull a repository and specify a branch add the __GIT_BRANCH__ environment variable:
```
docker run -d -e 'GIT_EMAIL=email_address' -e 'GIT_NAME=full_name' -e 'GIT_USERNAME=git_username' -e 'GIT_REPO=github.com/project' -e 'GIT_PERSONAL_TOKEN=<long_token_string_here>' -e 'GIT_BRANCH=stage' richarvey/nginx-php-fpm:latest
```
#### SSH keys

##### Preparing your SSH key
The container has the option for you to pass it the __SSH_KEY__ variable with a **base64** encoded private key. First generate your key and then make sure to add it to github and give it write permissions if you want to be able to push code from the container. Then run:
```
base64 -w 0 /path_to_your_key
```
**Note:** Copy the output, but be careful not to copy your prompt

##### Running with SSH Keys

To run the container and pull code simply specify the GIT_REPO URL including *git@* and then make sure you have also supplied your base64 version of your ssh deploy key:
```
sudo docker run -d -e 'GIT_NAME=full_name' -e 'GIT_USERNAME=git_username' -e 'GIT_REPO=github.com/project' -e 'SSH_KEY=BIG_LONG_BASE64_STRING_GOES_IN_HERE' richarvey/nginx-php-fpm:latest
```

To pull a repository and specify a branch add the GIT_BRANCH environment variable:
```
sudo docker run -d -e 'GIT_NAME=full_name' -e 'GIT_USERNAME=git_username' -e 'GIT_REPO=github.com/project' -e 'SSH_KEY=BIG_LONG_BASE64_STRING_GOES_IN_HERE' -e 'GIT_BRANCH=stage' richarvey/nginx-php-fpm:latest
```

### Custom Nginx Config files
Sometimes you need a custom config file for nginx to do rewrites or password protection, etc. For this reason we've included the ability to have custom nginx configs pulled directly from your git source. Please have a read of the [repo layout guidelines](docs/repo_layout.md) for more information. Its pretty simple to enable this, all you need to do is include a folder in the root of your repository called ```conf/nginx/``` within this folder you need to include a file called ```nginx-site.conf``` which will contain your default nginx site config. If you wish to have a custom file for SSL you simply include a file called ```nginx-site-ssl.conf``` in the same directory. These files will then be swapped in after you code is cloned.

### Scripting
There is often an occasion where you need to run a script on code to do a transformation once code lands in the container. For this reason we have developed scripting support. By including a scripts folder in your git repository and passing the __RUN_SCRIPTS=1__ flag to your command line the container will execute your scripts. Please see the [repo layout guidelines](https://github.com/ngineered/nginx-php-fpm/blob/master/docs/repo_layout.md) for more details on how to organise this.

### Lets Encrypt support (Experimental)
#### Enabling SSL or Special Nginx Configs
You can either map a local folder containing your configs to /etc/nginx or (recommended method) edit the files within the  __conf__ directory in a copy of this git repo, and then rebuilding the base image.
#### Setup
You can use Lets Encrypt to secure your container. Make sure you start the container with the ```DOMAIN, GIT_EMAIL``` and ```WEBROOT``` variables set to enable this functionality. Then run:
```
sudo docker exec -t <CONTAINER_NAME> /usr/bin/letsencrypt-setup
```
Ensure your container is accessible on the ```DOMAIN``` you supplied in order for this to work
#### Renewal
Lets Encrypt certs expire every 90 days, to renew simply run:
```
sudo docker exec -t <CONTAINER_NAME> /usr/bin/letsencrypt-renew
```

## Special Git Features
Specify the ```GIT_EMAIL``` and ```GIT_NAME``` variables for this to work. They are used to set up git correctly and allow the following commands to work.

### Push code to Git
To push code changes made within the container back to git run:
```
sudo docker exec -t -i <CONTAINER_NAME> /usr/bin/push
```

### Pull code from Git (Refresh)
In order to refresh the code in a container and pull newer code from git run:
```
sudo docker exec -t -i <CONTAINER_NAME> /usr/bin/pull
```

### Using environment variables

To set the variables pass them in as environment variables on the docker command line.

Example:
```
sudo docker run -d -e 'YOUR_VAR=VALUE' richarvey/nginx-php-fpm
```

## Logging and Errors

### Logging
All logs should now print out in stdout/stderr and are available via the docker logs command:
```
docker logs <CONTAINER_NAME>
```
### WebRoot
You can set your webroot in the container to anything you want using the ```WEBROOT``` variable e.g -e "WEBROOT=/var/www/html/public". By default code is checked out into /var/www/html/ so if your git repository does not have code in the root you'll need to use this variable.
