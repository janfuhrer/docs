# docs: kubernetes/redis
#kubernetes #kafka

## snippets
#snippets 

```bash
# get infos
redis-cli -h 127.0.0.1 -p 6379 info

# turn off the replication, turning the Redis server into a MASTER 
redis-cli -h 127.0.0.1 -p 6379 slaveof no one
```