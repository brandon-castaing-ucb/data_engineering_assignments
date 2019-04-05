## Change directory to assignment 5
  382  cd assignment-05-brandon-castaing-ucb/

## Check docker compose file for cluster configuration
  383  cat docker-compose.yml

## Spin up docker cluster containers
  384  docker-compose up

## Check 2 containers were created
  385  docker ps -a

## At this point, visited Jupyter Notebooks and began to interact with redis (Bonus portion)

## Spin down cluster
  386  docker-compose down
 
## Check all containers were destroyed
  387  docker ps -a

## Check history
  388  history

