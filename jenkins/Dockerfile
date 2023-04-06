FROM amazoncorretto:11
LABEL application="Jenkins"
LABEL author="Gopi"
ADD https://updates.jenkins.io/download/war/2.384/jenkins.war /jenkins.war
#ADD https://github.com/gopivurata/Ansible-playbooks/blob/main/jenkins/jenkins.service /jenkins.service
EXPOSE 8080
WORKDIR /
CMD [ "java", "-jar", "jenkins.war" ]