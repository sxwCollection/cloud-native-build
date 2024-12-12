# what
There are 2 parts in this demo.  
## first part
It is showing how to build a spring native image behind the firewall/proxy  
what will be done for the build?  
->  
1 the docker build image and the run image will be pulled by spring-maven-plugin    
2 pakete-buildpacks will run for instance in this demo the paketo-buildpacks/ca-certificates   
paketo-buildpacks/bellsoft-liberica  
paketo-buildpacks/syft              
paketo-buildpacks/executable-jar    
paketo-buildpacks/spring-boot       
and paketo-buildpacks/native-image    
are in use.    
3 the binaries of the dependencies which is defined in the buildpack.toml will be downloaded at build-time.    
4 the problem was the download in step 3 is blocked by cooperation firewall.    

# --> solution: download the binaries manually and configure them for buildpacks.
this could be done with dependency-mapping.  
the structure of the dependency-mapping should be:
1. the directories: bindings/dependency-mapping/    
2. the files in dependency-mappings dir: type with the content 'dependency-mapping' and the file with the name of sha-value and with the content of the binary location, this could be local like   
file:///bindings/dependency-mapping/binaries/xxx.tar.gz or some reachable URI  
3. another way to find the sha-value is build the project with logging level 'DEBUG',   
to configure it just set <plugin><artifactId>spring-boot-maven-plugin</artifactId><configuration><image><env><BP_LOG_LEVEL>DEBUG</BP_LOG_LEVEL>
and the sha-value could be shown in the loggings. 
3. you could download binaries and put them in the binaries directory  

after that bind the volume with maven plugin options: configuration/image: bindings/bind and env/SERVICE_BINDING_ROOT   
or with the pack command args --volume pathTo/bindings:/bindings --env SERVICE_BINDING_ROOT=/bindings  
at the end build with  
mvn -Pnative spring-boot:build-image    

# configurations
to config the buildpacks: in <plugin><artifactId>spring-boot-maven-plugin</artifactId><configuration><image><env>  
to configure the graalVM: in <plugin><artifactId>native-maven-plugin</artifactId><configuration><buildArgs>    
## second part
In this part it is shown how to start and debug native test.  
1. generate aot tests: mvn clean test spring-boot:process-test-aot. check the directory "target/spring-aot/test/sources"  
2. run the aot tests on JVM: 
   - set this @SpringBootApplication(proxyBeanMethods = false)    
   - add "target/spring-aot/test/sources" as test source root dir   
   - mvn -Dspring.aot.enabled=true test    
   - disable mockito tests because it is not supported    
   - disable tests if they are not supported   
   - @import runtimeHints if they are needed   
the native test starting looks like "Starting AOT-processed XXXTest"   
If GraalVm is installed, "mvn -PnativeTest test" could be used to run the native tests on GraalVm.  

