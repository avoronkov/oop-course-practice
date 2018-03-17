Семинар 6 (12.03.2018)
======================

build.xml for ant
-----------------

```
<project>
	<property name="src.dir"     value="src"/>
	<property name="build.dir"   value="build/"/>
	<property name="classes.dir" value="${build.dir}/classes"/>
	<property name="jar.dir"     value="${build.dir}/jar"/>

	<property name="test.src.dir"     value="src"/>
	<property name="test.build.dir" value="${build.dir}/test"/>


	<property name="jar.path"     value="${jar.dir}/Lab.jar"/>
	<property name="main.class"  value="Main" />

	<path id="classpath.test">
		<pathelement location="${user.home}/dev/java/junit/junit-4.12.jar"/>
		<pathelement location="${user.home}/dev/java/junit/hamcrest-core-1.3.jar"/>
		<pathelement location="${classes.dir}"/>
	</path>

    <target name="clean">
        <delete dir="build"/>
    </target>

    <target name="compile">
        <mkdir dir="${classes.dir}"/>
        <javac srcdir="${src.dir}" destdir="${classes.dir}">
			<classpath refid="classpath.test"/>
		</javac>
    </target>

	<target name="jar" depends="compile">
        <mkdir dir="${jar.dir}"/>
        <jar destfile="${jar.path}" basedir="${classes.dir}">
            <manifest>
                <attribute name="Main-Class" value="${main.class}"/>
            </manifest>
        </jar>
    </target>

	<target name="run" depends="jar">
        <java jar="${jar.path}" fork="true"/>
    </target>

	<target name="test-compile" depends="compile">
		<mkdir dir="${test.build.dir}"/>
		<javac srcdir="${test.src.dir}" destdir="${test.build.dir}" includeantruntime="false">
			<classpath refid="classpath.test"/>
		</javac>
	</target>

	<target name="test" depends="test-compile">
		<junit printsummary="on" haltonfailure="yes" fork="true">
			<classpath>
				<path refid="classpath.test"/>
				<pathelement location="${test.build.dir}"/>
			</classpath>
			<formatter type="brief" usefile="false" />
			<batchtest>
				<fileset dir="${test.src.dir}" includes="**/*Test.java" />
			</batchtest>
		</junit>
	</target>
</project>
```

Что необходимо поменять?
------------------------

Значение property `main.class` должно указывать на запускаемый класс, содержащий метод `main`.

Что должно работать?
--------------------

- `ant jar` дожен собирать jar-файл

- `ant test` должен собирать и запускать тесты.