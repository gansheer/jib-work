JIB experimentations
====================

Incremental images
------------------

### First image 

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven

#### Quarkus Jib extension

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-container-image-jib</artifactId>
</dependency>
```

mvn -e install -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2     --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml     -Dmaven.artifact.threads=12 -T 12 -Dquarkus.container-image.builder=jib -Dquarkus.container-image.push=true -Dquarkus.container-image.registry=localhost:5000 -Dquarkus.container-image.group=firstimage -Dquarkus.container-image.name=camel-k-integration  -Dquarkus.container-image.tag=latest -Dquarkus.container-image.insecure=true  -Dquarkus.jib.working-directory=/deployments/dependencies -Dquarkus.jib.base-jvm-image=eclipse-temurin:11

docker pull localhost:5000/firstimage/camel-k-integration
dive localhost:5000/firstimage/camel-k-integration

#### Jkube command

Add to pom:
```xml
      <plugin>
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>kubernetes-maven-plugin</artifactId>
        <version>1.10.1</version>
        <configuration>
          <buildStrategy>jib</buildStrategy>
          <images>
            <image>
              <name>localhost:5000/firstimage/${project.artifactId}:%l</name>
              <build>
                <from>eclipse-temurin:11</from>
                <assembly>
                  <mode>dir</mode>
                  <targetDir>/deployments</targetDir>
                  <layers>
                    <layer>
                      <id>dependencies</id>
                      <fileSets>
                        <fileSet>
                          <directory>${project.basedir}/../context/dependencies</directory>
                          <outputDirectory>dependencies</outputDirectory>
                        </fileSet>
                      </fileSets>
                    </layer>
                  </layers>
                  <excludeFinalOutputArtifact>true</excludeFinalOutputArtifact>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e k8s:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e k8s:push -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/firstimage/camel-k-integration:latest
dive localhost:5000/firstimage/camel-k-integration:latest


#### Jib plugin command

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven

Add to pom:
```xml
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.3.1</version>
        <dependencies>
          <dependency>
            <groupId>com.google.cloud.tools</groupId>
            <artifactId>jib-layer-filter-extension-maven</artifactId>
            <version>0.3.0</version>
          </dependency>
        </dependencies>
        <configuration>
          <to>
            <image>localhost:5000/firstimage/${project.artifactId}</image>
            <tags>
              <tag>latest</tag>
            </tags>
          </to>
          <from>
            <image>eclipse-temurin:11</image>
          </from>
          <container>
            <entrypoint>INHERIT</entrypoint>
            <args>
              <arg>jshell</arg>
            </args>
          </container>
          <allowInsecureRegistries>true</allowInsecureRegistries>

          <extraDirectories>
            <paths>
              <path>
                <from>../context</from>
                <into>/deployments</into>
              </path>
            </paths>
          </extraDirectories>
          <pluginExtensions>
            <pluginExtension>
              <implementation>com.google.cloud.tools.jib.maven.extension.layerfilter.JibLayerFilterExtension</implementation>
              <configuration implementation="com.google.cloud.tools.jib.maven.extension.layerfilter.Configuration">
                <filters>
                  <!-- Delete all jar files added automatically by jib. -->
                  <filter>
                    <glob>/app/**</glob>
                  </filter>
                </filters>
              </configuration>
            </pluginExtension>
          </pluginExtensions>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e jib:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/firstimage/camel-k-integration:latest
dive localhost:5000/firstimage/camel-k-integration:latest

#### Fabric8 docker plugin

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven


Add to pom:


```xml
      <plugin>
        <groupId>io.fabric8</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>0.41.0</version>
        <configuration>
          <images>
            <image>
              <name>localhost:5000/firstimage/${project.artifactId}:latest</name>
              <build>
                <from>eclipse-temurin:11</from>
                <assembly>
                  <name>deployments</name>
                  <inline>
                    <id>copy-dependencies</id>
                    <formats>
                      <format>dir</format>
                    </formats>
                    <fileSets>
                      <fileSet>
                          <outputDirectory>dependencies</outputDirectory>
                        <directory>${project.basedir}/../context/dependencies</directory>
                      </fileSet>
                    </fileSets>
                  </inline>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e clean package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:build -Ddocker.build.jib=true -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:push -Ddocker.build.jib=true -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/firstimage/camel-k-integration:latest
dive localhost:5000/firstimage/camel-k-integration:latest

### Second image 

cd /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven

#### Quarkus Jib extension

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-container-image-jib</artifactId>
</dependency>
```

mvn -e install -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2     --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml     -Dmaven.artifact.threads=12 -T 12 -Dquarkus.container-image.builder=jib -Dquarkus.container-image.push=true -Dquarkus.container-image.registry=localhost:5000 -Dquarkus.container-image.group=secondimage -Dquarkus.container-image.name=camel-k-integration  -Dquarkus.container-image.tag=latest -Dquarkus.container-image.insecure=true  -Dquarkus.jib.working-directory=/deployments/dependencies -Dquarkus.jib.base-jvm-image=localhost:5000/firstimage/camel-k-integration:latest

docker pull localhost:5000/secondimage/camel-k-integration
dive localhost:5000/secondimage/camel-k-integration

#### Jkube command

Add to pom:
```xml
      <plugin>
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>kubernetes-maven-plugin</artifactId>
        <version>1.10.1</version>
        <configuration>
          <buildStrategy>jib</buildStrategy>
          <images>
            <image>
              <name>localhost:5000/secondimage/${project.artifactId}:%l</name>
              <build>
                <from>localhost:5000/firstimage/camel-k-integration:latest</from>
                <assembly>
                  <mode>dir</mode>
                  <targetDir>/deployments</targetDir>
                  <layers>
                    <layer>
                      <id>dependencies</id>
                      <fileSets>
                        <fileSet>
                          <directory>${project.basedir}/../context/dependencies</directory>
                          <outputDirectory>dependencies</outputDirectory>
                        </fileSet>
                      </fileSets>
                    </layer>
                  </layers>
                  <excludeFinalOutputArtifact>true</excludeFinalOutputArtifact>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e k8s:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e k8s:push -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/secondimage/camel-k-integration:latest
dive localhost:5000/secondimage/camel-k-integration:latest

#### Jib plugin command

cd /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven

Add to pom:
```xml
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.3.1</version>
        <dependencies>
          <dependency>
            <groupId>com.google.cloud.tools</groupId>
            <artifactId>jib-layer-filter-extension-maven</artifactId>
            <version>0.3.0</version>
          </dependency>
        </dependencies>
        <configuration>
          <to>
            <image>localhost:5000/secondimage/${project.artifactId}</image>
            <tags>
              <tag>latest</tag>
            </tags>
          </to>
          <from>
            <image>localhost:5000/firstimage/camel-k-integration:latest</image>
          </from>
          <container>
            <entrypoint>INHERIT</entrypoint>
            <args>
              <arg>jshell</arg>
            </args>
          </container>
          <allowInsecureRegistries>true</allowInsecureRegistries>

          <extraDirectories>
            <paths>
              <path>
                <from>../context</from>
                <into>/deployments</into>
              </path>
            </paths>
          </extraDirectories>
          <pluginExtensions>
            <pluginExtension>
              <implementation>com.google.cloud.tools.jib.maven.extension.layerfilter.JibLayerFilterExtension</implementation>
              <configuration implementation="com.google.cloud.tools.jib.maven.extension.layerfilter.Configuration">
                <filters>
                  <!-- Delete all jar files added automatically by jib. -->
                  <filter>
                    <glob>/app/**</glob>
                  </filter>
                </filters>
              </configuration>
            </pluginExtension>
          </pluginExtensions>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e jib:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/secondimage/camel-k-integration:latest
dive localhost:5000/secondimage/camel-k-integration:latest

#### Fabric8 docker plugin

cd /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven

Add to pom:

```xml
      <plugin>
        <groupId>io.fabric8</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>0.41.0</version>
        <configuration>
          <images>
            <image>
              <name>localhost:5000/secondimage/${project.artifactId}:latest</name>
              <build>
                <from>localhost:5000/firstimage/camel-k-integration:latest</from>
                <assembly>
                  <name>deployments</name>
                  <inline>
                    <id>copy-dependencies</id>
                    <formats>
                      <format>dir</format>
                    </formats>
                    <fileSets>
                      <fileSet>
                          <outputDirectory>dependencies</outputDirectory>
                        <directory>${project.basedir}/../context/dependencies</directory>
                      </fileSet>
                    </fileSets>
                  </inline>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:build -Ddocker.build.jib=true -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:push -Ddocker.build.jib=true -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/second_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/second_image/extract_2/kit-cfl012bdm89s73c9irbg-2413910201/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/secondimage/camel-k-integration:latest
dive localhost:5000/secondimage/camel-k-integration:latest

### Third image 

cd /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven

#### Quarkus Jib extension

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-container-image-jib</artifactId>
</dependency>
```

mvn -e install -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2     --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml     -Dmaven.artifact.threads=12 -T 12 -Dquarkus.container-image.builder=jib -Dquarkus.container-image.push=true -Dquarkus.container-image.registry=localhost:5000 -Dquarkus.container-image.group=thirdimage -Dquarkus.container-image.name=camel-k-integration  -Dquarkus.container-image.tag=latest -Dquarkus.container-image.insecure=true  -Dquarkus.jib.working-directory=/deployments/dependencies -Dquarkus.jib.base-jvm-image=localhost:5000/secondimage/camel-k-integration:latest

docker pull localhost:5000/thirdimage/camel-k-integration
dive localhost:5000/thirdimage/camel-k-integration

#### Jkube command

Add to pom:
```xml
      <plugin>
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>kubernetes-maven-plugin</artifactId>
        <version>1.10.1</version>
        <configuration>
          <buildStrategy>jib</buildStrategy>
          <images>
            <image>
              <name>localhost:5000/thirdimage/${project.artifactId}:%l</name>
              <build>
                <from>localhost:5000/secondimage/camel-k-integration:latest</from>
                <assembly>
                  <mode>dir</mode>
                  <targetDir>/deployments</targetDir>
                  <layers>
                    <layer>
                      <id>dependencies</id>
                      <fileSets>
                        <fileSet>
                          <directory>${project.basedir}/../context/dependencies</directory>
                          <outputDirectory>dependencies</outputDirectory>
                        </fileSet>
                      </fileSets>
                    </layer>
                  </layers>
                  <excludeFinalOutputArtifact>true</excludeFinalOutputArtifact>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

```sh
# Package project
mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

# Build image
mvn -e k8s:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

# Push image
mvn -e k8s:push -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

# Verify image
docker pull localhost:5000/thirdimage/camel-k-integration:latest
dive localhost:5000/thirdimage/camel-k-integration:latest
```

#### Jib plugin command

cd /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven

```xml
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.3.1</version>
        <dependencies>
          <dependency>
            <groupId>com.google.cloud.tools</groupId>
            <artifactId>jib-layer-filter-extension-maven</artifactId>
            <version>0.3.0</version>
          </dependency>
        </dependencies>
        <configuration>
          <to>
            <image>localhost:5000/thirdimage/${project.artifactId}</image>
            <tags>
              <tag>latest</tag>
            </tags>
          </to>
          <from>
            <image>localhost:5000/secondimage/camel-k-integration:latest</image>
          </from>
          <container>
            <entrypoint>INHERIT</entrypoint>
            <args>
              <arg>jshell</arg>
            </args>
          </container>
          <allowInsecureRegistries>true</allowInsecureRegistries>

          <extraDirectories>
            <paths>
              <path>
                <from>../context</from>
                <into>/deployments</into>
              </path>
            </paths>
          </extraDirectories>
          <pluginExtensions>
            <pluginExtension>
              <implementation>com.google.cloud.tools.jib.maven.extension.layerfilter.JibLayerFilterExtension</implementation>
              <configuration implementation="com.google.cloud.tools.jib.maven.extension.layerfilter.Configuration">
                <filters>
                  <!-- Delete all jar files added automatically by jib. -->
                  <filter>
                    <glob>/app/**</glob>
                  </filter>
                </filters>
              </configuration>
            </pluginExtension>
          </pluginExtensions>
        </configuration>
      </plugin>
```

```sh
# Package project
mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

# Build and push image
mvn -e jib:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

# Verify image layers
docker pull localhost:5000/thirdimage/camel-k-integration:latest
dive localhost:5000/thirdimage/camel-k-integration:latest
```

#### Fabric8 docker plugin

cd /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven

Add to pom:

```xml
      <plugin>
        <groupId>io.fabric8</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>0.41.0</version>
        <configuration>
          <images>
            <image>
              <name>localhost:5000/thirdimage/${project.artifactId}:latest</name>
              <build>
                <from>localhost:5000/secondimage/camel-k-integration:latest</from>
                <assembly>
                  <name>deployments</name>
                  <inline>
                    <id>copy-dependencies</id>
                    <formats>
                      <format>dir</format>
                    </formats>
                    <fileSets>
                      <fileSet>
                          <outputDirectory>dependencies</outputDirectory>
                        <directory>${project.basedir}/../context/dependencies</directory>
                      </fileSet>
                    </fileSets>
                  </inline>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:build -Ddocker.build.jib=true -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:push -Ddocker.build.jib=true -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/third_image/extract_1/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/third_image/extract_1/kit-cfl025bdm89s73c9irc0-1277641253/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/thirdimage/camel-k-integration:latest
dive localhost:5000/thirdimage/camel-k-integration:latest


Multi-arch
----------

#### Quarkus jib extension

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven

Add to pom:
```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-container-image-jib</artifactId>
</dependency>
```

mvn -e install -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2     --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml     -Dmaven.artifact.threads=12 -T 12 -Dquarkus.container-image.builder=jib -Dquarkus.container-image.push=true -Dquarkus.container-image.registry=quay.io -Dquarkus.container-image.group=gfournie -Dquarkus.container-image.name=test  -Dquarkus.container-image.tag=multiarchquarkus -Dquarkus.container-image.insecure=true  -Dquarkus.jib.working-directory=/deployments/dependencies -Dquarkus.jib.base-jvm-image=eclipse-temurin:11 -Dquarkus.jib.platforms="linux/amd64,linux/arm64/v8"

Check : https://quay.io/repository/gfournie/test?tab=tags


#### JKube kubernetes plugin

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven


Add to pom:
```xml
      <plugin>
        <groupId>org.eclipse.jkube</groupId>
        <artifactId>kubernetes-maven-plugin</artifactId>
        <version>1.10.1</version>
        <configuration>
          <buildStrategy>jib</buildStrategy>
          <images>
            <image>
              <name>quary.io/gfournie/test:multiarchjkube</name>
              <build>
                <from>eclipse-temurin:11</from>
                <assembly>
                  <mode>dir</mode>
                  <targetDir>/deployments</targetDir>
                  <layers>
                    <layer>
                      <id>dependencies</id>
                      <fileSets>
                        <fileSet>
                          <directory>${project.basedir}/../context/dependencies</directory>
                          <outputDirectory>dependencies</outputDirectory>
                        </fileSet>
                      </fileSets>
                    </layer>
                  </layers>
                  <excludeFinalOutputArtifact>true</excludeFinalOutputArtifact>
                </assembly>
                <buildx>
                  <platforms>
                    <platform>linux/amd64</platform>
                    <platform>linux/arm64</platform>
                  </platforms>
                </buildx>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e k8s:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e k8s:push -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/firstimage/camel-k-integration:latest
dive localhost:5000/firstimage/camel-k-integration:latest


#### Fabric8 docker plugin

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven


Add to pom:


```xml
      <plugin>
        <groupId>io.fabric8</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>0.41.0</version>
        <configuration>
          <images>
            <image>
              <name>localhost:5000/firstimage/${project.artifactId}:latest</name>
              <build>
                <from>eclipse-temurin:11</from>
                <assembly>
                  <name>deployments</name>
                  <inline>
                    <id>copy-dependencies</id>
                    <formats>
                      <format>dir</format>
                    </formats>
                    <fileSets>
                      <fileSet>
                          <outputDirectory>dependencies</outputDirectory>
                        <directory>${project.basedir}/../context/dependencies</directory>
                      </fileSet>
                    </fileSets>
                  </inline>
                </assembly>
                <cmd>
                  <shell>jshell</shell>
                </cmd>
              </build>
            </image>
          </images>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e docker:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12 -Ddocker.build.jib=true

mvn -e k8s:push -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

docker pull localhost:5000/firstimage/camel-k-integration:latest
dive localhost:5000/firstimage/camel-k-integration:latest



#### Jib plugin command

cd /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven

Add to pom:
```xml
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>3.3.1</version>
        <dependencies>
          <dependency>
            <groupId>com.google.cloud.tools</groupId>
            <artifactId>jib-layer-filter-extension-maven</artifactId>
            <version>0.3.0</version>
          </dependency>
        </dependencies>
        <configuration>
          <to>
            <image>quay.io/gfournie/test</image>
            <tags>
              <tag>multiarch</tag>
            </tags>
          </to>
          <from>
            <image>eclipse-temurin:11</image>
            <platforms>
              <platform>
                <architecture>amd64</architecture>
                <os>linux</os>
              </platform>
              <platform>
                <architecture>arm64</architecture>
                <os>linux</os>
              </platform>
            </platforms>
          </from>
          <container>
            <entrypoint>INHERIT</entrypoint>
            <args>
              <arg>jshell</arg>
            </args>
          </container>
          <allowInsecureRegistries>true</allowInsecureRegistries>

          <extraDirectories>
            <paths>
              <path>
                <from>../context</from>
                <into>/deployments</into>
              </path>
            </paths>
          </extraDirectories>
          <pluginExtensions>
            <pluginExtension>
              <implementation>com.google.cloud.tools.jib.maven.extension.layerfilter.JibLayerFilterExtension</implementation>
              <configuration implementation="com.google.cloud.tools.jib.maven.extension.layerfilter.Configuration">
                <filters>
                  <!-- Delete all jar files added automatically by jib. -->
                  <filter>
                    <glob>/app/**</glob>
                  </filter>
                </filters>
              </configuration>
            </pluginExtension>
          </pluginExtensions>
        </configuration>
      </plugin>
```

mvn -e package -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

mvn -e jib:build -Dmaven.repo.local=/home/gfournie/work/documents/layers_jib/first_image/extract_2/artifacts/m2 --global-settings /home/gfournie/work/documents/layers_jib/first_image/extract_2/kit-cfkvvqjdm89s73c9irb0-351212210/maven/settings.xml -Dmaven.artifact.threads=12 -T 12

Check : https://quay.io/repository/gfournie/test?tab=tags


Notes
-----

### Quarkus jib extension

Does not allow build a layer from anything else than target/lib

### JKube kubernetes plugin

Does not allow any multi-arch config

### JIB maven plugin

Default build does : https://github.com/GoogleContainerTools/jib/blob/master/docs/faq.md#what-would-a-dockerfile-for-a-jib-built-image-look-like

What is the use of "<format>OCI</format>" ? (same image observed)

