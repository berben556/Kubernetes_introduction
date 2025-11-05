# Build stage
# Import de l'image du jdk 21 pour le build en donnant l'alias "myapp-build" à ce stage
FROM eclipse-temurin:21-jdk-alpine AS backend-build

# Déclaration d'une variable d'environnement  MYAPP_HOME=/opt/myapp
ENV BACKEND_HOME=/opt/backend

# Déclaration du répertoire de travail à partir de la variable déclaré
WORKDIR $BACKEND_HOME

# Installation de maven dans le container 
RUN apk add --no-cache maven

# Copy du pom xml dans le repertoire de travail
COPY pom.xml .

# Cache mvn dependencies entre les changements du pom.xml
RUN mvn dependency:go-offline

#Copy du répertoire src du projet maven dans un répertoire src dans le  workdir
COPY src ./src
# run de la commande d'e packaging de l'application mvn
RUN mvn package -DskipTests

# Run stage à partir d'un image de java runtime 
FROM eclipse-temurin:21-jre-alpine

#Même chose que dans le stage précédent
ENV BACKEND_HOME=/opt/backend
WORKDIR $BACKEND_HOME

# Copy des .jar générer par le build dans le stage précédent
COPY --from=backend-build $BACKEND_HOME/target/*.jar $BACKEND_HOME/myapp.jar

# Lancement du projet maven 
ENTRYPOINT ["java", "-jar", "myapp.jar"]
