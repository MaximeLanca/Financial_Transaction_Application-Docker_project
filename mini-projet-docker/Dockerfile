FROM amazoncorretto:17-alpine
LABEL maintainer="Maxime Lanca"
VOLUME /data
COPY target/paymybuddy.jar /paymybuddy.jar
EXPOSE 8080
CMD ["java", "-jar", "/paymybuddy.jar"]

