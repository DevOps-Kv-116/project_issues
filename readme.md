### Install docker
	sudo dnf update -y
	sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
	sudo dnf install -y docker-ce 
	sudo systemctl start docker
	sudo systemctl enable docker
	sudo groupadd docker
	sudo gpasswd -a $USER docker


### Install git
	sudo dnf install -y git

### Clone repos
	git clone --recurse-submodules https://github.com/DevOps-Kv-116/project_issues.git
	cd project_issues

### Add bridge network

	sudo docker network create --driver bridge --subnet 10.0.1.0/24 --ip-range 10.0.1.0/24 bridge_issue

### Build images
	docker build --tag="rest_api" rest-api/
	docker build --tag="rabbit_to_db" rabbit-to-db/
	docker build --tag="json_filter" json-filter/
	docker build --tag="frontend" frontend/
	docker build --tag="slack" rabbit_to_slack

### Run containers
Maybe you need to change some env vars for your instance

	docker run -h postgres --rm --name postgres --net bridge_issue -e POSTGRES_PASSWORD=Init1234 -e POSTGRES_HOST=postgres -e POSTGRES_USER=issueuser -e POSTGRES_PW=Init1234 -e POSTGRES_DB=issuedb -e USERMAP_UID=999 -e USERMAP_GID=999 -d -p 5432:5432 -v /home/projectIssues/docker/volumes/postgres:/var/lib/postgresql/data docker.io/library/postgres:latest

	docker run -h restapi --name restapi --net bridge_issue -d -p 5000:5000 -e github='https://github.com/DevOps-Kv-116/rest-api' -e POSTGRES_HOST=postgres -e POSTGRES_PORT=5432 -e POSTGRES_USER=issueuser -e POSTGRES_PW=Init1234 -e POSTGRES_DB=issuedb --rm rest_api

	docker run -d --rm --name rabbitmq -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=devops -e RABBITMQ_DEFAULT_PASS=softserve rabbitmq

	docker run -h rabbit_to_postgres --name rabbit_to_db --net bridge_issue -d -e POSTGRES_HOST=postgres -e POSTGRES_PORT=5432 -e POSTGRES_USER=issueuser -e POSTGRES_PW=Init1234 -e POSTGRES_DB=issuedb -e RABBIT_HOST=<Public IP>  -e RABBIT_PORT=5672 -e RABBIT_USER=devops -e RABBIT_PW=softserve -e RABBIT_QUEUE=restapi --rm rabbit_to_db

	docker run -h json_filter --name json_filter --net bridge_issue -d --rm -p 6000:5000 -e RMQ_HOST=<Public IP>  -e RMQ_PORT=5672 -e RMQ_LOGIN=devops -e RMQ_PASS=softserve -e QUEUE_SLACK=slack -e QUEUE_RESTAPI=restapi -e HOST=0.0.0.0 -e PORT=5000 --rm json_filter

	docker run -h frontend --name frontend --net bridge_issue -d -p 80:5000 -e POSTGRES_HOST=postgres -e POSTGRES_PORT=5432 -e POSTGRES_USER=issueuser -e POSTGRES_PW=Init1234 -e POSTGRES_DB=issuedb -e RESTAPI_HOST=<Public IP> -e RESTAPI_PORT=5000  --rm frontend

	docker run -h slack --name slack --net bridge_issue -d --rm -e RABBIT_HOST=<Public IP>  -e RABBIT_PORT=5672 -e RABBIT_USER=devops -e RABBIT_PW=softserve -e RABBIT_QUEUE=slack -e SLACK_URL=URL -e SLACK_CHANNEL=#chanel slack


In this artictle you can see how to create url for SLACK_URL variable [link](https://medium.com/@sharan.aadarsh/sending-notification-to-slack-using-python-8b71d4f622f3)

	
