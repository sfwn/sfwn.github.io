# maven download any artifact

> Date:  2018-06-06
>
> Author: sfwn

To download any artifact use

`mvn dependency:get -Dartifact=groupId:artifactId:version:packaging:classifier`

For Groovy sources this would be

`mvn dependency:get -Dartifact=org.codehaus.groovy:groovy-all:2.4.6:jar:sources`

For Groovy's javadoc you would use

`mvn dependency:get -Dartifact=org.codehaus.groovy:groovy-all:2.4.6:jar:javadoc`