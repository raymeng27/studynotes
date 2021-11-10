# Maven

## 一、关于Maven

### 1.官方网站：

https://maven.apache.org/

### 2.Maven仓库：

https://mvnrepository.com/

### 3.安装Maven

下载maven之后解压，然后设置环境变量：M2_HOME，然后将maven的bin目录加入到PATH环境变量中去。

### 4.目录结构：

bin：该目录包含了maven运行的脚本，这些脚本用来配置java命令。

boot：该目录只包含一个文件，plexus-classworlds是一个类加载器框架，相对于默认的java类加载器。它提供了更丰富的语法以方便配置，Maven使用该框架加载自己的类库。

conf：该目录包含了一个非常重要的文件settings.xml，直接修改该文件，就能在机器上全局的定制Maven的行为。

lib：该目录包含了所有Maven运行时需要的java类库。

LICENSE.txt

NOTICE.txt

README.txt

### 5.Tips

在IDEA或其它IDE中，当集成Maven时，都会安装上一个内嵌的Maven，这个内嵌的Maven通常会比较新，但不一定稳定。会出现两个潜在的问题：首先，较新的版本会存在很多不稳定因素，其次，除了IDE，也还会经常使用命令行的Maven，如果版本不一致，很容易造成构建的不一致。

## 二、坐标和依赖

### 1.坐标

Maven定义了这样一组规则：世界上仁和一个构建都可以使用Maven坐标唯一标识，Maven坐标的元素包括：groupId、artifactId、version、packaging、classifier。Maven内置了一个中央仓库的地址（https://repo1.maven.org/maven2），该中央仓库包含了世界上大部分流行的开源项目构建，Maven会在需要的时候去那里下载。

* groupId：定义当前Maven项目隶属的实际项目。groupId不应该对应项目隶属的组织或公司。groupId的表示方式与Java包名的标识方式类似，通常与域名反向一一对应。
* artifactId：该元素定义实际项目中的一个maven项目，推荐做法为使用实际项目名称作为artifactId的前缀。
* version：该元素定义Maven项目当前所处的版本。
* packaging：该元素定义Maven项目的打包方式。打包方式通常与所生成构件的文件拓展名对应，打包方式会影响到构件的生命周期，比如jar打包和war打包会使用不同的命令，当不定义packaging的时候，Maven会使用默认值jar。
* classifier：该元素用来帮助定义构建输出的一些附属构件，包含了Java文档和源代码，javadoc和sources就是这两个附属构件的classifier。不能直接定义项目的classifier，因为附属构件不是项目直接默认生成的，而是由附加的插件帮助生成的。

上述5个元素中，groupId、artifactId、version是必须定义的，packaging是可选的（默认为jar），classifier是不能直接定义的。

### 2.依赖

1. 依赖的配置

   根元素project下的dependencies可以包含一个或多个dependency元素，以声明一个或者多个项目依赖，每个项目依赖可以包含的元素有：

   * groupId、artifactId和version：依赖的基本坐标，基本坐标是最重要的，Maven根据坐标才能找到需要的依赖。
   * type：依赖的类型，对应于项目坐标定义的packaging。大部分不必声明，其默认值为jar。
   * scope：依赖的范围。
   * optional：标记依赖是否可选。
   * exclusions：用来排除传递性依赖。

2. 依赖范围

   首先需要知道，Maven在编译项目主代码的时候需要使用一套classpath，Maven在编译和执行测试的时候会使用另外一套classpath，实际运行Maven项目的时候，又会使用一套classpath。依赖范围（scope）就是用来控制与这三种classpath的关系，Maven有以下几种依赖范围：

   * compile：编译依赖范围，如果没有指定，就会默认使用该依赖范围，使用此依赖范围的Maven的依赖对于编译、测试、运行三种classpath都有效。

   * test：测试依赖范围，使用此Maven依赖，只对于测试classpath有效。

   * provided：已提供依赖范围，此依赖范围对于编译和测试classpath有效，但在运行时无效。

   * runtime：运行时依赖范围，对于测试和运行classpath有效，但在编译主代码时无效。

   * system：系统依赖范围。和provided依赖范围完全一致。但是使用syste依赖范围必须通过systemPath元素显示的指定依赖文件的路径。由于此类依赖不是通过maven仓库解析的，而且往往与本机系统绑定，可能造成构建的不可移植。例：

     ```xml
     <dependency>
     	<groupId>javax.sql</groupId>
         <artifactId>jdbc-stdext</artifactId>
         <version>2.0</version>
         <scope>system</scope>
         <systemPath>$JAVA_HOME/lib/rt.jar</systemPath>
     </dependency>
     ```

   * import(maven2.0.9及以上)：导入范围依赖，此依赖不会对三种classpath产生实际的影响。
   
3. 传递性依赖

   有了传递性依赖，在引用依赖的时候就不用考虑它依赖什么，也不用担心引入多余的依赖。Maven会解析各个直接依赖的POM，将那些必要的间接依赖，以传递性依赖的形式引入到当前项目。

   依赖范围不仅可以控制依赖与三种classpath的关系，还对传递性依赖产生影响。第一直接依赖的范围和第二直接依赖的范围决定了传递性依赖的范围，如下图，中间的交叉单元格表示传递性依赖范围。

   |          | compile  | test | provided | runtime  |
   | :------: | :------: | :--: | :------: | :------: |
   | compile  | compile  |  --  |    --    | runtime  |
   |   test   |   test   |  --  |    --    |   test   |
   | provided | provided |  --  | provided | provided |
   | runtime  | runtime  |  --  |    --    | runtime  |

4. 依赖调解

   例如，项目A有这样的依赖关系：A -> B -> C -> X(1.0)、A -> D -> X(2.0)，X是A的传递性依赖，但是两条路径上有两个版本的X。Maven依赖调解的第一原则是：路径最近者优先。

   但是如果，依赖路径长度是一样的，比如：A -> B -> X(1.0)、A -> D -> X(2.0)，在Maven2.0.8之前，这是不确定的，但是从Maven2.0.9开始，Maven定义了依赖调解的第二原则：第一声明者优先。

5. 可选依赖

   依赖中的标签<optional>元素表示这个依赖为可选依赖，它们对当前项目产生影响，当其他项目依赖于此项目的时候，这个可选依赖不会被传递。如下：

   ```xml
   <dependencies>
   	<dependency>
       	<groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
   		<version>8.0.21</version>
           <optional>true</optional>
       </dependency>
   </dependencies>
   ```

6. 排除依赖

   在<dependency>标签中使用<exclusions>元素声明排除依赖，<exclusions>包含一个或多个<exclusion>子元素，因此可以排除一个或多个传递性依赖。需要注意的是，声明<exclusion>的时候只需要gruopId和artifactId就行，而不使用version元素，因为只需要gruopId和artifactId就能在唯一定位依赖图中的某个依赖。

7. 归类依赖

   如果多个依赖使用相同版本的值，则可以在<properties>元素中定义Maven属性，然后在依赖中引入定义的变量值即可，如${spring.version}。代码如下：

   ```xml
   <properties>
   	<spring.version>5.3.0</spring.version>
   </properties>
   <dependencies>
   	<dependency>
       	<groupId>org.springframework</groupId>
           <artifactId>spring-core</artifactId>
           <version>${spring.version}</version>
       </dependency>
   </dependencies>
   ```

## 三、仓库

### 1.何为仓库

坐标和依赖是任何一个构建在Maven世界中的逻辑标识方式；而构建的物理表示方式是文件，maven通过仓库来统一管理这些文件。得益于坐标机制，任何maven项目使用任何一个构件的方式都是完全相同的，在此基础上，maven可以在某个位置统一存储所有maven项目共享的构件，这个统一的位置就是仓库。

### 2.仓库的布局

任何一个构件都有其唯一的坐标，根据这个坐标可以定义其在仓库中的唯一存储路径，这边是maven的仓库布局方式。例如，log4j:log4j:1.2.15这一依赖，其对应的仓库路径为log4j/log4j/1.2.15/log4j-1.2.15.jar，该路径与坐标的大致对应关系为groupId/artifactId/version/artifactId-version.packaging。

### 3.仓库的分类

对于maven来说，仓库只分为两类：本地仓库和远程仓库。

* 本地仓库

  默认情况下，不管是在Windows还是Linux上，每个用户在自己的用户目录下都有一个路径为.m2/repository/的仓库目录。用户也可以自己定义本地仓库目录地址。可以编辑文件~/.m2/settings.xml，设置localRepository元素的值为想要的仓库路径：

  ```xml
  <settings>
  	<localRepository>/home/yourepositorydir</localRepository>
  </settings>
  ```

  需要注意的是，默认情况下，~/.m2/settings.xml文件是不存在的，用户需要从maven安装目录复制settings.xml文件再进行编辑。一个构件只有在本地仓库中之后，才能由其它maven项目使用，那么构件如何进入到本地仓库中呢？最常见的是依赖maven从远程仓库下载到本地仓库中。还有一种常见的就是将本地项目的构件安装到maven仓库中，在某个项目中执行mvn clean install命令。install插件的install目标将项目的构建输出文件安装到本地仓库。

* 远程仓库

  安装好maven后，不执行任何命令，本地仓库目录是不存在的。当用户输入第一条maven命令后，maven才会创建本地仓库，然后根据配置和需要，从远程仓库下载构建至本地仓库。对于maven来说，每个用户只有一个本地仓库，但可以配置访问很多远程仓库。

* 中央仓库

  https://repo1.maven.org/maven2/

* 私服

  私服是一种特殊的远程仓库，它是假设在局域网内的仓库服务，私服代理广域网上的远程仓库，供局域网内的maven用户使用。当maven需要下载构件的时候，它从私服请求，如果私服上不存在该结构，则从外部的远程仓库下载，缓存在私服上之后，再为maven的下载请求提供服务。

  私服可以帮助你：节省自己的外网宽带、加速构建maven、部署第三方构件、提高稳定性，增强控制、降低中央仓库的负荷。

### 4.远程仓库的配置

很多情况下，默认的中央仓库无法满足项目的需求，可能项目需要的构件存在于另外一个远程仓库中，这时，可以在POM中配置该仓库。代码：

```xml
<project>
	<repositories>
    	<repository>
        	<id>jboss</id>
            <name>JBoss Repository</name>
            <url>http://repository.jboss.com/maven2/</url>
            <releases>
            	<enabled>true</enabled>
            </releases>
            <snapshots>
            	<enabled>false</enabled>
            </snapshots>
            <layout>default</layout>
        </repository>
    </repositories>
</project>
```

在repositories元素下，可以使用repository子元素声明一个或多个远程仓库。在该例中声明一个id为Jboss，名称为Jboss repository的仓库。任何一个仓库声明的id必须是唯一的，Maven自带的中央仓库使用的id为central，如果其它的仓库声明也使用id，就会覆盖中央仓库的配置。该配置的url为指向了仓库的地址。在配置中的releases和snapshots元素用来控制maven对于发布版构件和快照版构件的下载。layout元素值default表示仓库的布局是maven2及maven3的默认布局，而不是maven1的布局。

对于releases和snapshots，除了enabled，还包括另外两个子元素updatePolicy和checksumPolicy：

```xml
<snapshots>
	<enabled>true</enabled>
    <updatePolicy>daily</updatePolicy>
    <checksumPlicy>ignore</checksumPlicy>
</snapshots>
```

元素updatePolicy用来配置maven从远程仓库检查更新的频率，默认值daily，表示一天，never--从不检查更新，always--每次构建都会检查更新，interval：X--每隔X分组检查一次。checksumPolicy用来配置Maven检查检验和文件的策略。如果校验和验证失败默认值为warn时，Maven会在构建是输出警告信息，fail时Maven会构建失败，ignore使Maven完全忽略校验和错误。

### 5.仓库搜索服务

* Sonatype Nexus

  http://repository.sonatype.org/

* Jarvana

  http://www.jarvana.com/jarvana/

* MVNbrowser

  http://www.mvnbrowser.com

* MVNrepository

  http://mvnrepository.com/

## 三、生命周期和插件

### 1.生命周期

Maven的生命周期就是为了对所有的构建过程进行抽象和统一。这个生命周期包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建步骤。Maven的生命周期是抽象的，本身并不做任何实际的工作，在Maven的设计中，实际的任务都交由插件来完成。Maven每个构建步骤都可以绑定一个或者多个插件行为，而且Maven为大多数构建步骤编写并绑定了默认插件。

Maven拥有三套相互独立的生命周期，分别为clean、default和site。clean生命周期的目的是清理项目，default生命周期的目的是构建项目，而site的生命周期的目的是建立项目站点。每个周期包含一些阶段，这些阶段是有顺序的，并且后面的阶段依赖前面的阶段，三套生命周期是相互独立的。

#### 1.clean生命周期

clean的生命周期的目的是清理项目，包含三个阶段：

* pre-clean：执行一些清理前需要完成的工作。
* clean：清理上一次构建生成的文件。
* post-clean：执行一些清理后需要完成的工作。

#### 2.default生命周期

default生命周期定义了真正构建时所需要执行的所有步骤，它是所有生命周期中最核心的部分，包含的阶段有：

validate、initialize、generate-sources、process-sources、generate-resources、process-resources、compile、process-classes、generate-test-sources、process-test-sources、generate-test-resources、process-test-resources、test-compile、process-test-classes、test、prepare-package、package、pre-integration-test、integration-test、post-integration-test、verify、install、deploy

#### 3.site生命周期

site生命周期的目的是建立和发布项目站点，Maven能够基于POM所包含的信息，自动生成一个友好的站点，发布项目信息，包含阶段如下：

* pre-site：执行一些在生成项目站点之前需要完成的工作。
* site：生成项目站点文档。
* post-site：执行一些在生成项目站点之后需要完成的工作。
* site-deploy：将生成的项目站点发布到服务器上。

### 2.插件

#### 1.插件目标

Maven的核心仅仅定义了抽象的声明周期，具体的任务是交由插件完成的，插件以独立的构件形式存在，为了能够复用代码，它往往能够完成多个任务。每个插件会聚集多个功能，每个功能就是一个插件目标。

#### 2.插件绑定

Maven的生命周期与插件相互绑定，用以完成实际的构建任务。具体而言，是生命周期的阶段与插件的目标相互绑定，以完成某个具体的构建任务。