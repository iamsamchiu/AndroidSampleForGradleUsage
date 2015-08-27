# AndroidSampleForGradleUsage

The aim of the project is to demonstrate some good usage of gradle.

#### Customs versionCode/versionName generation schema in build.gradle for multiple APK
- /app/build.gradle:
```java
apply plugin: 'com.android.application'

ext{
    minSdkVersion = 19
    versionMajor = 1
    versionMinor = 0
    buildNumber = 1
    phone_flavor_type = 1
    tablet_flavor_type = 2
}

android {
    compileSdkVersion 22
    buildToolsVersion "23.0.0"

    defaultConfig {
        applicationId "com.samtest.myandroidapp"
        minSdkVersion "${project.ext.minSdkVersion}"
        targetSdkVersion 22
        versionCode 1
        versionName computeVersionName()
    }
    buildTypes {
        debug {
            buildConfigField "boolean","NETWORK_DEBUG","true"
            minifyEnabled false
        }
        release {
            buildConfigField "boolean","NETWORK_DEBUG","false"
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    productFlavors {
        phone{
            versionCode computeVersionCode(project.ext.phone_flavor_type)
        }
        tablet{
            versionCode computeVersionCode(project.ext.tablet_flavor_type)
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.2.1'
    compile 'com.google.android.gms:play-services:7.5.0'
    //compile 'com.google.android.gms:play-services-base:7.5.0'
}

def computeVersionCode(int flavor) {
    def version_code = ext.minSdkVersion * 100000 + flavor * 1000 + ext.versionMajor * 100 + ext.versionMinor
    return version_code
}
def computeVersionName(){
    ext.buildNumber = System.getenv("BUILD_NUMBER") ?: "dev"
    def version_name = ext.versionMajor+"."+ext.versionMinor+"-"+ext.buildNumber
    return version_name
}

```


#### Lib-project could use consumerProguardFiles property to specify his proguard configuration file.
- /mylibrary/build.gradle:
```groovy
    buildTypes {
        release {
            consumerProguardFiles 'proguard-rules.pro'
        }
    }

```


#### Sample task to publish AAR library to maven from a lib-project.
- /mylibrary/uploadMaven.gradle:
``` groovy
apply plugin: 'maven-publish'

publishing {
    publications {
        repositories.maven {
            url "${project.getProperty('systemProp.maven.serverHost')}"
            credentials {
                username "${project.getProperty('systemProp.maven.userName')}"
                password "${project.getProperty('systemProp.maven.passWord')}"
            }
        }
        maven(MavenPublication) {
            groupId "com.samtest.myandroidapp"
            artifactId project.archivesBaseName
            version "1.0.0"
            pom.withXml {
				        //add dependencies info to support transitive dependency management
            	  def dependenciesNode = asNode().appendNode('dependencies')
            	  configurations.compile.allDependencies.each {
                    if(it instanceof org.gradle.api.internal.artifacts.dependencies.DefaultExternalModuleDependency){
                        def dependencyNode = dependenciesNode.appendNode('dependency')
                        dependencyNode.appendNode('groupId', it.group)
                        dependencyNode.appendNode('artifactId', it.name)
                        dependencyNode.appendNode('version', it.version)
                    }
                 }
                 //SCM info is an integral part of any healthy project. If your lib-project uses an SCM system,
                 //you could provide it when upload artifect to Maven.
                 def scm = asNode().appendNode('scm')
                 scm.appendNode('url', 'https://github.com/iamsamchiu/ReadmePlugin')
                 scm.appendNode('connection', 'scm:https://github.com/iamsamchiu/ReadmePlugin.git')
                 scm.appendNode('developerConnection', 'scm:ssh://git@github.com/iamsamchiu/ReadmePlugin.git'')
                 scm.appendNode('tag', version)
            }
        }
    }
}
```
