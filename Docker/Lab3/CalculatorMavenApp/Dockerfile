FROM maven:3.9-eclipse-temurin-17

RUN useradd -m appuser
WORKDIR /app
RUN chown appuser:appuser /app
USER appuser

COPY --chown=appuser:appuser pom.xml .
COPY --chown=appuser:appuser src ./src
RUN mvn package

EXPOSE 8080
CMD ["java", "-jar", "target/calculator.jar"]
