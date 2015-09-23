# docker-lita

This repository is used to build a base Docker image for deploying Lita: [litaio/lita](https://hub.docker.com/r/litaio/lita/). Note that the tags of this image on the Docker Hub refer to the version of Ruby used. The version of Lita used is specified in your own Gemfile.

## Usage

Configure your Lita instance to connect to Redis on the "redis" host:

``` ruby
Lita.configure do |config|
  config.redis[:host] = "redis"
end
```

No other Redis configuration is needed.

Add a `Dockerfile` with the following contents to the root directory of your Lita instance's repository:

``` Dockerfile
FROM litaio/lita
```

That's all it needs!

Build the image with this command from the root of the repository:

``` bash
docker build -t $LITA_IMAGE_NAME .
```

Replace `$LITA_IMAGE_NAME` with a custom name for your Lita image.

Before starting a Lita container, make sure a Redis container is running:

``` bash
docker run -d --name redis --restart always -v $REDIS_PATH:/var/lib/redis litaio/redis
```

Replace `$REDIS_PATH` with the absolute path to the directory on the host machine where Redis's data should be stored.

Now start the Lita container:

``` bash
docker run -d --name lita --link redis --restart always -v $BUNDLE_PATH:/var/bundle -p 80:$LITA_HTTP_PORT $LITA_IMAGE_NAME
```

Replace `$BUNDLE_PATH` with the absolute path to the directory on the host machine where gems should be cached. Replace `$LITA_HTTP_PORT` with whichever port you've configured for your Lita instance's HTTP server (the default is 8080 if you haven't set it explicitly). Replace `$LITA_IMAGE_NAME` with the name you chose when you built the image earlier.

Lita is now running!

To deploy a new version of your bot, commit or pull any changes to the repository, rerun the command to build the image, then stop, remove, and start the Lita container again:

~~~ bash
docker build -t $LITA_IMAGE_NAME .
docker stop lita
docker rm lita
docker run -d --name lita --link redis --restart always -v $BUNDLE_PATH:/var/bundle -p 80:$LITA_HTTP_PORT $LITA_IMAGE_NAME
~~~

## License

[MIT](http://opensource.org/licenses/MIT)
