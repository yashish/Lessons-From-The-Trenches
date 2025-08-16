To create and run RabbitMQ in Docker desktop locally, we can use either of these methods

## How to Create RabbitMQ Docker Container Locally

1. Ensure Docker is installed and running on your machine.
2. Open a terminal and run:

	```cmd
	docker run -d --name rabbitmq -p 8080:5672 -p 15672:15672 rabbitmq:management
	```

	- This pulls the RabbitMQ image with the management plugin, starts it in detached mode, and exposes ports 8080 (messaging) and 15672 (management UI).

3. Access the RabbitMQ Management UI at [http://localhost:15672](http://localhost:15672) (default username/password: guest/guest).

## Alternate Method: Using Docker Compose

You can use Docker Compose to run RabbitMQ easily. This is useful for managing multiple services or customizing configuration.

1. Create a `docker-compose.yml` file (already provided in this directory):

		```
		version: '3.8'
		services:
			rabbitmq:
				image: rabbitmq:management
				container_name: rabbitmq
				ports:
					- "8080:5672"
					- "15672:15672"
				environment:
					RABBITMQ_DEFAULT_USER: guest
					RABBITMQ_DEFAULT_PASS: guest
		```

2. In the same directory, run:

		```
		docker compose up -d
		```

3. Access the RabbitMQ Management UI at [http://localhost:15672](http://localhost:15672) (default username/password: guest/guest).

Running the docker compose command in detached mode
<img width="1729" height="482" alt="image" src="https://github.com/user-attachments/assets/30b1619f-c5d8-460c-b067-225055605342" />

<img width="1261" height="670" alt="image" src="https://github.com/user-attachments/assets/036f88c7-f9b4-4052-8c6c-bdcfe032b15f" />



