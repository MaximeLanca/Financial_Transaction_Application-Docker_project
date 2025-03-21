FROM amazoncorretto:17-alpine
LABEL maintainer="Maxime Lanca"
RUN apt update
VOLUME /data
COPY /root/mini-projet-docker/target/paymybuddy.jar /
EXPOSE 8080
CMD ["java", "-jar", "./paymybuddy.jar"]

