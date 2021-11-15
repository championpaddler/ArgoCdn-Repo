

### ArgoCDN

Objectives:

To build a GeoRedirector and caching system to server data from nearest server


Steps:

We are constructing two servers overs here one in us and other in India.

We upload data in usa and all data get cached on India location and users in asia region are served from India server. For us region and outside asia region we
server using USA Server.

USA server: http://argocdn.tk
India Server: http://in.argocdn.tk/

I have taken domain from freenom.com. 

Let's start :

1. Build the georedirector

For building it , we will load the maxmind geolocation binaries which will provide us the country code. We will be installing the full nginx having all support libraries.


Downloading the binaries 

```
sudo apt-get update
sudo apt-get install nginx-full 
wget -N https://github.com/mbcc2006/GeoLiteCity-data/blob/master/GeoLiteCity.dat
mv GeoLiteCity.dat /usr/share/GeoIP/
```


Loading the binaries in /etc/nginx/nginx.conf:

```
http {
  geoip_city /usr/share/GeoIP/GeoLiteCity.dat;
  ...
}
```


Now we will edit our virtual host (/etc/nginx/sites-available/default) to handle the redirection and redirect to nearest server.

```

// We will map the nearest server using the continent code like for Asia we have as and for other we are using default


map $geoip_city_continent_code $closest_server {
  default argocdn.tk;
  AS      in.argocdn.tk;
}

server {


      // Adding the available server name
      server_name  argocdn.tk
              in.argocdn.tk;
              
        // Redirection to closest server    
        if ($closest_server != $host) {
                rewrite ^ $scheme://$closest_server$request_uri break;
        }


}

```

Reload the Nginx and we are done.


2. Now building the caching system. This is done only on the secondary servers (primary server).

```

//Defining the proxy catch path along with parameters:

proxy_cache_path /var/www/cache levels=1:2 keys_zone=cdn:10m inactive=60m;

server {

 location / {
        proxy_cache cdn;
    // Defining the cache key
    proxy_cache_key $uri$is_args$args;
    proxy_cache_valid 90d;
    proxy_pass http://argocdn.tk;
        }
}

```

Testing of application :

You can test the application using vpn.

Try opening  http://argocdn.tk/argo.jpg you will be redirected to http://in.argocdn.tk/argo.jpg . You can also see caching in action as first time it takes 1 sec and
in subsequent request it takes 0.5 sec.

I am attaching both of files for references.




