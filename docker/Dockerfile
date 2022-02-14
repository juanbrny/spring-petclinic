FROM registry.suse.com/bci/openjdk:11-11.1

ARG ARTIFACT_NAME
COPY target/${ARTIFACT_NAME}.jar application.jar

CMD java -Dserver.port=${SERVER_PORT:-}\
          -Dserver.context-path=/petclinic/\
          -Dspring.messages.basename=messages/messages\
          -Dlogging.level.org.springframework=${LOG_LEVEL:-INFO}\
          -Dsecurity.ignored=${SECURITY_IGNORED:-/**}\
          -Dbasic.authentication.enabled=${AUTHENTICATION_ENABLED:-false}\
          -Dserver.address=${SERVER_ADDRESS:-0.0.0.0}\
          -jar application.jar
