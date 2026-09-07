cp = .:./lib/esper-common-8.2.0.jar:./lib/esper-runtime-8.2.0.jar:./lib/esper-compiler-8.2.0.jar:./lib/slf4j-api-1.7.25.jar:./lib/slf4j-jdk14-1.7.25.jar:./lib/antlr4-runtime-4.7.1.jar:./lib/janino-3.0.10.jar:./lib/commons-compiler-3.0.10.jar

all: clean
	javac WeatherTuple.java
	javac -classpath $(cp) Assignment1.java

run1:
	java -classpath $(cp) Assignment1 query_1.epl
run2:
	java -classpath $(cp) Assignment1 query_2.epl
run3:
	java -classpath $(cp) Assignment1 query_3.epl
run4:
	java -classpath $(cp) Assignment1 query_4.epl
run6:
	java -classpath $(cp) Assignment1 query_6.epl
run7:
	java -classpath $(cp) Assignment1 query_7.epl
run8:
	java -classpath $(cp) Assignment1 query_8.epl

clean:
	rm -f *.class
