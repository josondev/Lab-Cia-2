# Campus Equipment Booking Service
Java 17 / Spring Boot REST API for a CI/CD lab assessment.

## Build and tests
Run the Maven package build and the unit tests with:

```powershell
mvn clean package
mvn test
```

To run the application directly, use `mvn spring-boot:run`; it listens on port 8080.

## Endpoints
- `GET /api/equipment`
- `GET /api/equipment/{id}`
- `POST /api/bookings`
- `GET /api/bookings/{id}`
- `POST /api/bookings/{id}/cancel`
- `GET /actuator/health`

Inventory is in memory and resets on restart. No database is required.

## Docker
Build the required image and start the container:

```powershell
docker build -t campus-equipment-booking:set-d .
docker run -d -p 8080:8080 --name equipment-booking campus-equipment-booking:set-d
curl.exe http://localhost:8080/actuator/health
```

The health endpoint should return a JSON status of `UP`. If rerunning the container, remove the previous one first with `docker rm -f equipment-booking`.

## Minikube deployment
Start Minikube, load the local image, and apply the Deployment and NodePort Service:

```powershell
minikube start
minikube image load campus-equipment-booking:set-d
kubectl apply -f deployment.yaml
kubectl rollout status deployment/equipment-booking-deployment
kubectl get deployments
kubectl get pods
```

Expose the Service and test its health endpoint. Keep the terminal running `minikube service` open; on Windows with the Docker driver it maintains the local tunnel. Append `/actuator/health` to the URL printed by Minikube:

```powershell
minikube service equipment-booking-service --url
curl.exe http://127.0.0.1:<PORT>/actuator/health
```

Scale the Deployment to two replicas and verify the Pods:

```powershell
kubectl scale deployment equipment-booking-deployment --replicas=2
kubectl rollout status deployment/equipment-booking-deployment
kubectl get pods -l app=equipment-booking
```

## Validation behavior
The inventory boundary check rejects a request only when its quantity is greater than the available quantity. Booking exactly the remaining inventory is valid; requesting one more unit is rejected and leaves inventory unchanged. `exactInventoryBookingSuccess` and `exactPlusOneOverbookingRejected` cover these boundary cases.

Borrower identifiers are required to be non-null and non-blank. `CreateBookingRequest` applies `@NotBlank` during API request validation, and `BookingService.create` repeats the guard for direct service calls. The service throws `IllegalArgumentException` for a blank identifier; the tests cover empty and whitespace-only values.

## Jenkins
The root `Jenkinsfile` runs `mvn clean verify`, builds `campus-equipment-booking:set-d`, starts Minikube, applies the manifest, and scales the Deployment to two replicas. Configure the Jenkins agent with Java 17, Maven, Docker access, Minikube, and `kubectl`. The agent must use a Minikube profile and Docker engine that it can access; a Jenkins service account may not automatically share an interactive user's cluster profile.
