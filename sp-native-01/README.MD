In this demo it is shown how to build a spring native image behind the firewall/proxy  
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

--> solution: download the binaries manually and configure them for buildpacks.    
this could be done with dependency-mapping.  
the structure of the dependency-mapping should be:    
1 the directories: bindings/dependency-mapping/    
2 the files in dependency-mappings dir: type with the content 'dependency-mapping' and the file with the name of sha-value and with the content of the binary location, this could be local like  
file:///bindings/dependency-mapping/binaries/xxx.tar.gz or some reachable URI  
3 you could download binaries and put them in the binaries directory  

after that bind the volume with maven plugin options: configuration/image: bindings/bind and env/SERVICE_BINDING_ROOT   
or with the pack command args --volume pathTo/bindings:/bindings --env SERVICE_BINDING_ROOT=/bindings  
at the end build with  
mvn -Pnative spring-boot:build-image    
