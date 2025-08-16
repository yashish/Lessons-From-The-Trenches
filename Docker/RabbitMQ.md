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

		```yaml
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

		```cmd
		docker compose up -d
		```

3. Access the RabbitMQ Management UI at [http://localhost:15672](http://localhost:15672) (default username/password: guest/guest).