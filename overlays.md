# Apache Maven WAR Plugin -- Overlays #

Overlays are used to share common resources across multiple web applications. The dependencies of a WAR project are collected in **WEB-INF/lib**, except for WAR artifacts which are overlayed on the WAR project itself.

**Depend on another WAR artifact**

	...
	<dependency>
		<groupId>com.example.projects</groupId>
		<artifactId>doucumentedprojectdependency</artifactId>
		<version>1.0-SNAPSHOT</version>
		<type>WAR</type>
		<scope>runtime</scope>
	</dependency>
	...

The <overlay> element can have the following child elements:

- id - the id of the overlay. If none is provided, the WAR Plugin will generate one.
- groupId - the groupId of the overlay artifact you want to configure.
- artifactId - the artifactId of the overlay artifact you want to configure.
- type - the type of the overlay artifact you want to configure. Default value is: war.
- classifier - the classifier of the overlay artifact you want to configure if multiple artifacts matches the groupId/artifactId.
- includes - the files to include. By default, all files are included.
- excludes - the files to exclude. By default, the META-INF directory is excluded.
- targetPath - the target relative path in the webapp structure, which is only available for overlays of type war. By default, the content of the overlay is added in the root structure of the webapp.
- skip - set to true to skip this overlay. Default value is: false.
- filtered - whether to apply filtering to this overlay. Default value is false.



**Overlays are applied with a first-win strategy (hence if a file has been copied by one overlay, it won't be copied by another). ** Overlays are applied in the order in which they are defined in the <overlays> configuration. If no configuration is provided, the order in which the dependencies are defined in the POM is used.