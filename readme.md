# Build

* For config-service, discovery-server, ratings, reviews, details, use mvn to run that
~~~~~bash
mvn clean package
~~~~~
It will generated springboot fat *jar* file in each dir targets subdir

* For productpage, it's written in python, no need to build

Then run the following command to build docker images:
```bash
./docker-build-push.sh 
```

As the services has startup dependencies, you need to deploy it one by one following the strict sequence:

```shell
kubectl apply -f discovery-server.yaml
kubectl apply -f clickhouse.yaml
```

Then check the running status and logs to ensure the discovery server starts successfully and is UP.

`kubectl apply -f config-service.yaml`

Then check the running status and logs to ensure the config server starts successfully and is UP.

After that deploy the sample services and Ingress.

```shell
kubectl apply -f bookinfo.yaml
kubectl apply -f ingress.yaml
```

Take a note of your **Ingress public IP**, and the Ingress listens on port 8080 by default.

# Run

## start rating service: 
~~~~~bash
java -jar bookinfo-ratings-1.0.0-SNAPSHOT.jar --opentracing.jaeger.http-sender.url=http://jaeger-collector.default.svc:14268/api/traces
~~~~~
In VM environment, it will listen on localhost:8101. 

create rating in VM, visit the service directly:
~~~~~bash
curl -X POST http://localhost:8101/ratings \
	-H "Content-Type: application/json" \
	-d '{"reviewerId":"9bc908be-0717-4eab-bb51-ea14f669ef20","productId":"a071c269-369c-4f79-be03-6a41f27d6b5f","rating":3}' 
~~~~~

Through API Gateway:
~~~~~bash
curl -X POST http://localhost:10000/bookinfo-ratings/ratings \
	-H "Content-Type: application/json" \
	-d '{"reviewerId":"9bc908be-0717-4eab-bb51-ea14f669ef20","productId":"a071c269-369c-4f79-be03-6a41f27d6b5f","rating":3}' 
~~~~~  

or, create ratings in k8s, replace the ***ingress-ip*** with a real IP address:
~~~~~bash
curl -X POST http://ingress-ip:8080/bookinfo-ratings/ratings \
	-H "Content-Type: application/json" \
	-d '{"reviewerId":"9bc908be-0717-4eab-bb51-ea14f669ef20","productId":"a071c269-369c-4f79-be03-6a41f27d6b5f","rating":3}' 
~~~~~

query ratings by product_id in vm:
~~~~~bash
curl http://localhost:8101/ratings/a071c269-369c-4f79-be03-6a41f27d6b5f
curl http://localhost:10000/bookinfo-ratings/ratings/a071c269-369c-4f79-be03-6a41f27d6b5f
~~~~~

or, query ratings by product_id in kubernetes, replace the ***ingress-ip*** with a real IP address:
~~~~~bash
curl http://ingress-ip:8080/bookinfo-ratings/ratings/a071c269-369c-4f79-be03-6a41f27d6b5f
~~~~~

## start review service:
~~~~~bash
java -jar bookinfo-reviews-1.0.0-SNAPSHOT.jar --bookinfo-ratings.url=localhost:8101 --opentracing.jaeger.http-sender.url=http://jaeger-collector.default.svc:14268/api/traces
~~~~~
It will listen on localhost:8102 and will call localhost:8101 for rating query in vm environment.

or, start review service in k8s:
~~~~~bash
java -jar bookinfo-reviews-1.0.0-SNAPSHOT.jar --bookinfo-ratings.url=bookinfo-ratings.default.svc:9080 --opentracing.jaeger.http-sender.url=http://jaeger-collector.default.svc:14268/api/traces
~~~~~

create review in VM:
~~~~~bash
curl -X POST http://localhost:8102/reviews \
	-H "Content-Type: application/json" \
	-d '{"reviewerId":"9bc908be-0717-4eab-bb51-ea14f669ef20","productId":"a071c269-369c-4f79-be03-6a41f27d6b5f","review":"This was OK.","rating":3}' 
	
curl -X POST http://localhost:10000/bookinfo-reviews/reviews \
	-H "Content-Type: application/json" \
	-d '{"reviewerId":"9bc908be-0717-4eab-bb51-ea14f669ef20","productId":"a071c269-369c-4f79-be03-6a41f27d6b5f","review":"This was OK.","rating":3}'
~~~~~

or, create review in k8s, replace the ***ingress-ip*** with a real IP address:
~~~~~bash
curl -X POST http://ingress-ip:8080/bookinfo-reviews/reviews \
	-H "Content-Type: application/json" \
	-d '{"reviewerId":"9bc908be-0717-4eab-bb51-ea14f669ef20","productId":"a071c269-369c-4f79-be03-6a41f27d6b5f","review":"This was OK.","rating":3}'
~~~~~

query review by product_id in VM:
~~~~~bash
curl http://localhost:8102/reviews/a071c269-369c-4f79-be03-6a41f27d6b5f

curl http://localhost:10000/bookinfo-reviews/reviews/a071c269-369c-4f79-be03-6a41f27d6b5f
~~~~~

or, query review by product_id in k8s, replace the ***ingress-ip*** with a real IP address:
~~~~~bash
curl http://ingress-ip:8080/bookinfo-reviews/reviews/a071c269-369c-4f79-be03-6a41f27d6b5f
~~~~~

## start detail service:
~~~~~bash
java -jar bookinfo-details-1.0.0-SNAPSHOT.jar --opentracing.jaeger.http-sender.url=http://jaeger-collector.default.svc:14268/api/traces
~~~~~
It will listen on localhost:8103

query detail by isbn:
~~~~~bash
curl http://localhost:8103/details/1234567890

curl http://localhost:10000/bookinfo-details/details/1234567890
~~~~~

or, query detail by isbn in k8s, replace the ***ingress-ip*** with a real IP address:
~~~~~bash
curl http://ingress-ip:8080/bookinfo-details/details/1234567890
~~~~~


