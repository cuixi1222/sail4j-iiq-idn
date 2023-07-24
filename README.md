Use Sail4j to develop IdentityIQ or IdentityNow Rules
================================
# About Sail4j
Sail4j is a tool to convert Java code to Beanshell code used by IdentityIQ or IdentityNow Rules. This way of development improves the development efficiency and code quality through the following:
- eliminate Rule syntax error. 
- pave the way to adopt TDD (Test Driven Development) methodolgy by using JUnit, Mockito and other popular Java unit testing tools.

# Key features
- Sail4j uses Java annotation to describe the characters (such as name, rule type, file name and dependent rule libraries) of Rule that is developed in a standard Java class.

	Refer to the below section for how to **Use Sail4j to develop Rule**:

- Sail4j then converts the Java classes to Rule XML files via one of following 2 integration components you choose to use:
	- **Custom Apache Ant Task**
	
		Integreate with SSB (Standard Service Build, the latest version is v7.0.1 at time of this writing). This typically applies to an IdentityIQ project as most IdentityIQ implementation projects use SSB.
		
		Refer to the below section for how to **Configure Sail4j in SSB**:

	- **Maven Plugin**
		
		Integrate with Maven. This can be useful in a scenario where you already have a project hosting all the source code related to you IdentityIQ or IdentityNow installation, and you just need another simple java project where you can write and unit test your *Rules*. Once *Rules* are tested and generated by Sail4j, you will copy the XML files to your original project.
		
		Refer to the below section for how to **Setup a Maven project to use Sail4j**:


# Configure Sail4j in SSB

- Download SSB package from the following SailPoint Compass page:

		https://community.sailpoint.com/t5/Professional-Services/Services-Standard-Build-SSB-v7-0-1/ta-p/190496


- Create a new folder **lib/sail4j** under the SSB folder and copy the following jar files to this folder:
  
  

 		commons-collections-3.2.jar
 		slf4j-api-1.6.1.jar
 		javaparser-core-3.18.0.jar
		velocity-1.6.2.jar
 		velocity-tools-2.0.jar

 		sail4j-ant-task-1.1.jar
 		sail4j-api-1.1.jar
 		sail4j-transform-1.1.jar
		sail4j-test-helper-1.1
 		
	Note: the folders *s**ail4j-iiq-idn/sail4j-bundle*** and ***sail4j-iiq-idn/dependency-jars*** inside this repository include these jar files or you can download them from internet.

- When using Sail4j, you may want to write JUnit Test Cases to test your Java code. Create a new folder **lib/junit** and copy the following jar files to this folder:
	

 		junit.jar
 		mockito-core-2.23.4.jar
 		org.hamcrest.core_1.3.0.v20180420-1519.jar
	Note: the folder ***sail4j-iiq-idn/dependency-jars*** inside this repository include these jar files or you can download them from internet.

- When using Sail4j, you may want to write JUnit Test Cases to test your Java code. Add the following to ***scripts/build.java.xml***.

	<target name="runUnitTests" depends="compile">
		<javac srcdir="test-src" source="1.8" target="1.8" destdir="test/classes" debug="true" 	classpathref="build.compile.classpath" 
			includeantruntime="last"/>
		
		<!-- run junit tests -->
    	<junit printsummary="yes" haltonfailure="yes">
    		<classpath>
				<pathelement location="${build}/classes" />
    			<pathelement location="test/classes" />
    			<path refid="build.compile.classpath"/>
    			<fileset dir="lib/junit">
	            		<include name="*.jar"/>
	          	</fileset>
			</classpath>
    		<formatter type="plain" />
	        <formatter type="xml" />
	        <batchtest todir="test/report">
	            <fileset dir="test-src">
	                <include name="**/*Test*.java" />
	            </fileset>
	        </batchtest>
        </junit>
    </target>

- Update the ***main*** target of **build.xml** file to add the following 2 sections:

	The following section should be added right after `<antcall inheritall="true" target="compile"/>`. It is configured to run JUnit test cases. It also allows you to skip junit tests by passing parameter 'skipUnitTest' as true in the command. 

		<!-- run junit tests -->
    	<if>
    	    <not>
    	       <equals arg1="${skipUnitTest}" arg2="true"/>
    	     </not>
    	    <then>
    	       <antcall inheritall="true" target="runUnitTests"/>  	
    	    </then>
    	</if>


	The following section should be added right before `<antcall inheritall="true" target="prepareCustomConfig"/>`. It is to configure how the Rule XML files are generated. You can specify the location of Java source files and the folder where the Rule XML files will be generated.

       <!-- generate Rules from Java code -->
       <taskdef name="genRule" classname="com.sailpoint.sail4j.ant.GenerateRuleTask">
	     <classpath>
	      <fileset dir="lib/sail4j">
	        <include name="*.jar"/>
	      </fileset>
	      <fileset dir="lib">
	        <include name="*.jar"/>
	      </fileset>
	     </classpath>
	   </taskdef>
       <genRule sourceFilePath="src/sail4j/iam/rule" destinationXmlFilePath="config/Rule"/>
    	
- Update the ***war*** target of **build.xml** file to add the following line at the beginning:
  
  This line is to remove java classes (which are used to generate Rule XMLs) from the IdentityIQ application as these java classes are only used to generate Rules and should not be shipped into the IIQ application. It is recommended to use some special package names (such as sail4j in this example) for these types of Jave classes to make it easy to remove them.

		<!-- exclude the java classes used by Sail4j to generate Ruels -->
    	<delete dir="${build.iiqBinaryExtract}/WEB-INF/classes/sail4j"/>


# Setup Maven to use Sail4j

### Install Apache Maven

Download Apache Maven 3.6.x or higher from [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi). Installation is simply extracting the compressed archive, setting the M2_HOME environment variable and adding the $M2_HOME/bin to your $PATH. For example in Mac, just modify the file **.bash_profile** to add the Maven installation folder to PATH as below:

 		export M2_HOME=/Users/bruce.ren/Desktop/tools-install/apache-maven-3.6.3
		...
		export PATH=$PATH:$ANT_HOME/bin:$M2_HOME/bin

**Please note:** *currently if you use maven 3.9.x, you will experience error when executing the script install-iiq-jars.sh (refer to the sections below). You are recommmended to use 3.8.x before we resolve this issue. Also the scripts provided are shell scripts supported by Linux or Mac OS. If your OS is Windows, consider use WSL to run the shell scripts (or you may have other workaround).*
                                
### Install IdentityIQ and dependencies jars into local Maven repository
- Download the IdentityIQ base GA (e.g. **IdentityIQ 8.3.zip**) from Compass. The version of IIQ doesn't matter because it is only used to compile your java code which will be eventually converted to IIQ or IDN Rules.

- Open the file **sail4j-iiq-idn/mvn-install/install-iiq-jars.sh** to modify the following 2 lines to match the version and location of IdentityIQ you just download:

 		export IIQ_VERSION=8.3
 		export BASE_SOFTWARE_PATH=/Users/bruce.ren/Desktop/tools-install/iiq-install/base	            	
- Go to folder **sail4j-iiq-idn/mvn-install** to run the **install-iiq-jars.sh**, it takes a few minutes to install all the jar files.

		./install-iiq-jars.sh

### Install Sail4j jars into local Maven repository
- Modify the file **sail4j-iiq-idn/sail4j-bundle/sail4j-test-helper.pom.xml** to update the version of IdentityIQ configured in the previous step:

		<properties>
    		<IdentityIQ.Version>8.3</IdentityIQ.Version>
  		</properties>

- Go to folder **sail4j-iiq-idn/mvn-install** to run the **install-sail4j-jars.sh**.

		./install-sail4j-jars.sh


### Create Maven project
- Now you can create a Maven project by using the template project included in the following folder:

       sail4j-iiq-idn/maven-template

- Modify pom.xml to update groupId and artifactId if necessary. Please note this folder also includes 2 examples (with corresponding Junit test cases) for the reference.
- Import the project directory into Eclipse (or Intellij) as a Maven project.

# Use Sail4j to develop Rule
## Sail4j Annotations
You develop your IIQ or IDN Rule in a standard Java class but use Sail4j annotations to describe Rule. 

Annotations are used to control how the rule should be generated. There are 3 annotations available for use:
- **SailPointRule**
  
  Class level annotation. You use this to define the rule name, rule type, referencedRules etc.
- **SailPointRuleMainBody**
  
  Method level annotation. Use this annotation to indicate the code inside this method will be used as the main body of the Rule.
- **IgnoredBySailPointRule**
  
  Method and field level annotation. Use this annotation to indicate the field or method should not be included when the Rule is generated.

Here is an example:

 		package com.demo.rule;
 		
		import org.apache.commons.lang.StringUtils;
		import org.apache.commons.logging.Log;
		import org.apache.commons.logging.LogFactory;
		
		import com.sailpoint.sail4j.annotation.IgnoredBySailPointRule;
		import com.sailpoint.sail4j.annotation.SailPointRule;
		import com.sailpoint.sail4j.annotation.SailPointRule.RuleType;
		import com.sailpoint.sail4j.annotation.SailPointRuleMainBody;
	
		import sailpoint.api.SailPointContext;
		import sailpoint.connector.ConnectorException;
		import sailpoint.object.Application;
		import sailpoint.object.Identity;
		import sailpoint.tools.GeneralException;

		@SailPointRule(name = "ActiveDirectorySAMAccountName", type = RuleType.FIELD_VALUE, 
			referencedRules = {"Rule-Library-General", "Rule-Library-ActiveDirectory-Search"})
		public class ActiveDirectorySAMAccountNameFieldValueRule {
	
			private static Log ruleLog = LogFactory.getLog("rule.library.active.directory");
			
			@IgnoredBySailPointRule
			private String notUsed;
			
			@IgnoredBySailPointRule
			//Dummy method. The real method is actually inside one of referenced rule libraries.
			public boolean userNameExistInActiveDirectory(SailPointContext context, Identity identity, Application application,
					String strUserName) throws GeneralException, ConnectorException {
				return false;
			}
		
			//Construct will be ignored when generating Rule
			public ActiveDirectorySAMAccountNameFieldValueRule() {
				
			}
		
			public String getActiveDirectoryUniqueName(SailPointContext context, Identity identity,
					Application adApplication, boolean escapeName) throws GeneralException, ConnectorException {
				ruleLog.debug("=== Generate Active Directory Unique username ===");
				
				String strUserName = "";
				try {
					String strFirstName = identity.getFirstname();
					String strLastName = identity.getLastname();
					
					// Code to handle unusual case of names are blank
					if (StringUtils.isEmpty(strFirstName) || StringUtils.isEmpty(strLastName)) {
						ruleLog.debug("=== strFirstName and or strLastName is null/empty ===");
						return "";
					}
		
					// Code to Replace special characters from the last name
					if (!StringUtils.isBlank(strLastName)) {
						strLastName = strLastName.replaceAll("[^a-zA-Z0-9]+", "").toUpperCase();
					}
					if (!StringUtils.isBlank(strFirstName)) {
						strFirstName = strFirstName.replaceAll("[^a-zA-Z0-9]+", "").toUpperCase();
					}
					
					// Code to get the first initial of First Name in a variable
					String strFirstNameSub = StringUtils.substring(strFirstName, 0, 1);
				
					String strLastNameSub = "";
		
					strUserName = strLastName + strFirstNameSub;
					ruleLog.debug("adAccountName created initially: " + strUserName);
		
					int iLength = StringUtils.length(strUserName);
					ruleLog.debug("Length of the adAccountName created initially: " + iLength);
					// Code to get the basic adAccountName
					if (iLength > 8) {
						strLastNameSub = StringUtils.substring(strLastName, 0, 8 - iLength);
						ruleLog.debug("strLastNameSub 8-iLength" + strLastNameSub);
						strUserName = strLastNameSub + strFirstNameSub;
						ruleLog.debug("adAccountName created after checking length: " + strUserName);
					}
					do {
						// Check if the calculated username exists in AD
						if (!userNameExistInActiveDirectory(context, identity, adApplication, strUserName)) {
							break;
						}
						// Check the numeric value added to create the unique adAccountName
						String strUser = strUserName.replaceAll("[^0-9]", "");
						int iNumber = 0;
						if (!StringUtils.isBlank(strUser)) {
							iNumber = Integer.valueOf(strUser).intValue();
						}
						int iLengthNumber = String.valueOf(iNumber + 1).length();
						// Create the adAccountName by adding numeric values to make it generic
						strLastNameSub = StringUtils.substring(strLastName, 0, 7 - iLengthNumber);
						strUserName = strLastNameSub + strFirstNameSub;
						iNumber++;
						strUserName += iNumber;
						ruleLog.debug("adAccountName generated with numbers after duplicate found: " + strUserName);
							
					} while (true);
					
					ruleLog.debug("Final User ID created for the new Identity: " + strUserName);
				} catch (Exception ex) {
					ruleLog.error("Exception while calcualting Identity adAccountName: ", ex);
					return "";
				}
					return strUserName;
				}
				
				@SailPointRuleMainBody
				public String executeRule(SailPointContext context, Identity identity, Application application) 
						throws GeneralException, ConnectorException {
				    String userName = getActiveDirectoryUniqueName(context, identity, application, true);
				   
					return userName;
				}
			}

## Generate Rule XMLs
When you run SSB command (`build.sh` or `ant main`) or Maven build command (`mvn install` or `mvn package`), Sail4j will covert the Java classes to Rules based on the values specified in the annotations (such as Rule name, Rule type, Rule file name, dependent rule library etc.). If it passes all the JUnit Tests, Rule XMLs will be genereated under the folder that is configured.

## Example source code
The folder ***sail4j-iiq-idn/maven-template*** contains a couple of examples helping you learn:
- how to write Rule in Java class using Sail4j;
- how to write JUnit Test by using Mockito to simulate IIQ/IDN API.


# Why Use Sail4j

Using Sail4j to develop Rule provides a few benefits. Here are the 2 main ones:
- The Rule will be guaranteed to be syntax error free. Jave IDE such as Eclipse or Intellij will do the syntax check for you when you develop your Rules.
- It opens the door to write unit test code to test your Rules when they are written in Java. There are many testing libraries and frameworks in the Java open-source ecosystem that you can use for unit testing. For example, you can use JUnit and Mockito to write unit test cases and simulate IdentityIQ or IdentityNow API. 