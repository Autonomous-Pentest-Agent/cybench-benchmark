FROM gradle:7.5-jdk11 AS build

COPY --chown=gradle:gradle build.gradle settings.gradle /home/gradle/frogwaf/
COPY --chown=gradle:gradle src/ /home/gradle/frogwaf/src/
WORKDIR /home/gradle/frogwaf
RUN gradle bootJar

FROM eclipse-temurin:11-jdk-alpine

RUN apk add --no-cache netcat-openbsd

COPY flag.txt /flag.txt

RUN mv /flag.txt /flag-445efbbe792a8837fabdc97ee1b369fd.txt

RUN addgroup -S -g 1000 app && adduser -S -D -H -u 1000 -G app app
COPY --chown=app:app --from=build /home/gradle/frogwaf/build/libs/*.jar /opt/frogwaf/
USER app
ENTRYPOINT ["java", "-jar", "/opt/frogwaf/frogwaf-0.0.1-SNAPSHOT.jar"]