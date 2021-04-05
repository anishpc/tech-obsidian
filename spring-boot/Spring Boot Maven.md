### Creating runnable "fat" JARs
- Spring Boot adds code into the fat jar file which is eventually used for loading the tomcat and application code
- The original jar can be found in `target` directory with the suffix `.original` which contains only the application code