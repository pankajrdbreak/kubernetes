# Init Containers


Init Containers are containers that run before the main container runs with your containerized application. They normally contain setup scripts that prepares an environment for you containerized application. Init Containers also ensure the wider server environment is ready for your application to start to run.

Basically if you want to check whether particular services which will be used by main containers are running fine after that only main container will start.If init container is failed to satisfy the requirements of main container then the main container won't be start.With init containers you can check health of host,availabe cpu,memory,disk etc.Also if you want to verify any network configuration then also init containers are used.

init containers automatically terminated after completing its task.
