

## Start the server locally

```bash
docker run -it -p 4000:4000 -v $(pwd):/tmp/app -w /tmp/app ruby:2.3.3 bundle install && /bin/bash
jekyll build --watch
```
