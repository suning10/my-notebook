    ## Docker 

### Docker Run
 - pull a new image 

```bash
docker run <image-name> <command>
```


### Docker Exec 
- this need a exsiting container 
- to run redis-command 
- use sh to avoid keep running docker exec
  - docker exec \<container id> sh
- docker compose exec
  - The difference is how you address the container: docker exec takes a container name or ID, while docker compose exec takes a service name from your **docker-compose.yml.**

```bash
docker exec <option> <container> <command>
docker exec -it redis redis-cli
i means interactive / t means ttl print to terminal
```
