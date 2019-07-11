# docker-compose_laradep
Laravel deployment using Docker Compose.

### First, let's setup our Laravel Application 
```
~$ laravel new laravel-app
~$ cd laravel-app
```

### Setting up a Ephimeral container to setup all the applications on the same image. 

```
~$ docker run --rm -v $(pwd):/App composer Install 
```

Now we have to setup some files on our sudo
