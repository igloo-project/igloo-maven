# Igloo : Maven & NPM integration

To trigger NPM build in a child module, add the following items in your `properties` and  `build` sections,
given your build generate contents in `src/main/generated-js/`, and that you create a
`src/main/generated-js/.gitkeep` empty file:

```xml
<properties>
	<frontend.npm.arguments>run js</frontend.npm.arguments>
</properties>
<build>
	<resources>
		<resource>
			<directory>${project.basedir}/src/main/generated-js/</directory>
			<targetPath>META-INF/resources/webjars/${project.name}/${project.version}</targetPath>
			<excludes>
				<exclude>.gitkeep</exclude>
			</excludes>
		</resource>
		<resource>
			<directory>src/main/webjars</directory>
			<targetPath>META-INF/resources/webjars/${project.name}/${project.version}</targetPath>
		</resource>
	</resources>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-clean-plugin</artifactId>
			<executions>
				<execution>
					<id>clean-node-modules</id>
					<phase>clean</phase>
				</execution>
			</executions>
		</plugin>
		<plugin>
			<groupId>com.github.eirslett</groupId>
			<artifactId>frontend-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```


The purpose of this fragment is :

* to enable `frontend-maven-plugin` that performs node and npm bootstrap, installs dependencies with `npm ci`,
  then triggers your npm build
* to setup arguments passed to npm command with `frontend.npm.arguments` property
* to package `src/main/generated-js/` in webjar location
* to trigger a custom clean task that take care of emptying `src/main/generated-js/` and `node_modules`

.gitkeep file must be pushed on repository, by adding `!.gitkeep` in your `.gitignore` file.

## package-lock.json

Dependencies are installed with `npm ci`, which installs the exact versions locked in `package-lock.json`
and fails if this file is missing or does not match `package.json`. This makes builds reproducible and
prevents a newly published (possibly compromised) version of a transitive dependency from being pulled
silently into the build.

`package-lock.json` must therefore be committed, next to `package.json`:

* remove `package-lock.json` from your `.gitignore` file, if present
* run `npm install` in the module, and commit the generated `package-lock.json`
* after any change to `package.json`, run `npm install` again and commit both files