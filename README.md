# BoardgameListingWebApp

## Description

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven

## CI/CD Diagram


![1711256345809](https://github.com/user-attachments/assets/687167f3-16bb-408c-a7c1-00780c8f4e68)

## Description
 - Developed a CI/CD pipeline using Jenkins for automated image builds, testing, and registry pushes.
 - Utilized Azure AKS for the Kubernetes cluster.
 - Created a Dockerfile for containerizing the application.
 - Deployed the Nginx Ingress Controller and configured DNS.
 - Enabled HTTPS using CERT_MANAGER and applied SSL/TLS certificates.
 - Followed GitOps practices with the use of ArgoCD.
 - Configured Horizontal Pod Autoscaler (HPA) to increase pod count when memory utilization crosses 50%.
