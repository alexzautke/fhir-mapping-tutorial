# FHIR tutorial mapping example
FHIR has a mapping language to convert between resources and logical models and features a [tutorial](http://build.fhir.org/mapping-tutorial.html) how use it. This projects tries to build the structure definitions and mappings for it.

See the [FHIR Mapping Language confluence page](https://confluence.hl7.org/display/FHIR/Using+the+FHIR+Mapping+Language) for additional information. 

To run the transforms directly from the command line you can use the FHIR Java Validator, [download here](https://github.com/hapifhir/org.hl7.fhir.core/releases/latest/download/validator_cli.jar).
For more information about the .NET Implementation, [see here](https://docs.fire.ly/mappingengine/index.html).

If you want to use it with a public test server, you can use https://test.ahdis.ch/r4/. Install [Visual Studio Code](https://code.visualstudio.com/) with the [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) extension. in each step there is a test.ahdis.ch.http file which you can upload the StructureDefinition and StructureMap and perform the transformation. E.g. see http file for [step1](https://github.com/ahdis/fhir-mapping-tutorial/blob/master/maptutorial/step1/test.ahdis.ch.http).

## run the tutorial
for each step there is a directory below the maptutorial directory

logical: structurdefinitions for tleft, tright

map: Mapping files in FHIR Mapping Language

source: XML files used to transform

output.xml: result of map transform

## Comparison Java Reference Implementation / .NET Reference Implementation

|          | Java | .NET |
|----------|------|------|
| Step 1   |      |   ✓  |
| Step 2   |      |   ✓  |
| Step 3a  |      |   ✓  |
| Step 3b  |      |   ✓  |
| Step 3c  |      |   ✓  |
| Step 4a  |      |   ✓  |
| Step 4b  |      |   ✗  |
| Step 4b2 |      |   ✓  |
| Step 4b3 |      |   ✓  |
| Step 4c  |      |   ✓  |
| Step 5a  |      |   ✓  |
| Step 5b  |      |   ✓  |
| Step 6a  |      |   ✓  |
| Step 6b  |      |   ✓  |
| Step 6c  |      |   ✓  |
| Step 6d  |      |   ✓  |
| Step 7a  |      |   ✓  |
| Step 7b  |      |   ✓  |
| Step 8   |      |   ✓ (Not using embedded ConceptMaps)   |
| Step 9   |      |   ✓  |
| Step 10  |      |   ✗  |
| Step 11  |      |      |
| Step 12  |      |      |
| Step 13  |      |      |
| Step 14  |      |      |
| Step 15  |      |      |


## [step1](http://hl7.org/fhir/mapping-tutorial.html#step1) - ok


```
java -jar validator_cli.jar ./maptutorial/step1/source/source1.xml -transform http://hl7.org/fhir/StructureMap/tutorial -version 4.0.1 -ig ./maptutorial/step1/logical -ig ./maptutorial/step1/map -log test.txt -output ./maptutorial/step1/output.xml
```

TLeft --> TRight

```xml
<TRight 
  xmlns="http://hl7.org/fhir/tutorial">
  <a value="step1"/>
</TRight>
```

Note: The short version does not work: src.a -> tgt.a, needs to be src.a as a -> tgt.a = a

## [step2](http://hl7.org/fhir/mapping-tutorial.html#step2) - ok

```
java -jar validator_cli.jar ./maptutorial/step2/source/source2.xml -transform http://hl7.org/fhir/StructureMap/tutorial -version 4.0.1 -ig ./maptutorial/step2/logical -ig ./maptutorial/step2/map -log test.txt -output ./maptutorial/step2/output.xml
```

TLeft --> TRight

```xml
<TRight xmlns="http://hl7.org/fhir/tutorial">
  <a2>test</a2>
</TRight>
```

## [step3](http://hl7.org/fhir/mapping-tutorial.html#step3)

note: did not use the structure definition like in steps1 and steps2 but simplified that the value is an attribute and not the content of the element.


### step3a

```
java -jar validator_cli.jar ./maptutorial/step3/source/source3.xml -transform http://hl7.org/fhir/StructureMap/tutorial3a -version 4.0.1 -ig ./maptutorial/step3/logical -ig ./maptutorial/step3/map -log test.txt -output ./maptutorial/step3/output.xml
```

just cut it off at 20 characters

```xml
<TRight xmlns="http://hl7.org/fhir/tutorial">
  <a2 value="01234567890123456789"/>
</TRight>
```

### step3b ignore it - fails

```
  source.a2 as a where a2.length()<20 -> target.a2 = a "rule_a20b";
```

change it to one of the following to run the map:

```
  source.a2 as a where length()<20 -> target.a2 = a "rule_a20b";
  source.a2 as a where $this.length()<20 -> target.a2 = a "rule_a20b";
  source.a2 as a where source.a2.length()<20 -> target.a2 = a "rule_a20b";
  source.a2 as a where %source.a2.length()<20 -> target.a2 = a "rule_a20b";

```

```
java -jar validator_cli.jar ./maptutorial/step3/source/source3.xml -transform http://hl7.org/fhir/StructureMap/tutorial3b -version 4.0.1 -ig ./maptutorial/step3/logical -ig ./maptutorial/step3/map -log test.txt -output ./maptutorial/step3/output.xml
```

### step3c fails: error if it's longer than 20 characters

```
rule_a20c: for source.a2 as a check a2.length() <= 20 make target.a2 = a
```

change it to one of the following to run the map:

```
rule_a20c: for source.a2 as a check $this.length() <= 20 make target.a2 = a
rule_a20c: for source.a2 as a check length() <= 20 make target.a2 = a
rule_a20c: for source.a2 as a check source.a2.length() <= 20 make target.a2 = a
```

```
java -jar validator_cli.jar ./maptutorial/step3/source/source3.xml -transform http://hl7.org/fhir/StructureMap/tutorial3c -version 4.0.1 -ig ./maptutorial/step3/logical -ig ./maptutorial/step3/map -log test.txt -output ./maptutorial/step3/output.xml
```


## [step4](http://hl7.org/fhir/mapping-tutorial.html#step4) - fails

```
src.a21 as a -> tgt.a21 = cast(a, "integer"); // error if it's not an integer
src.a21 as a where a.isInteger -> tgt.a21 = cast(a, "integer"); // ignore it
src.a21 as a where not at1.isInteger -> tgt.a21 = 0; // just assign it 0
```

- Rule "rule_a21a": cast to integer not yet supported
- Rule "rule_a21b": Error in step4b.map at 7, 8: The name isInteger is not a valid function name

changed it to:

```
  source.a21 as a where $this.matches('\\d+') -> target.a21 = a "rule_a21b";
```

## [step5](http://hl7.org/fhir/mapping-tutorial.html#step5) - ok 

```
java -jar validator_cli.jar ./maptutorial/step5/source/source5b.xml -transform http://hl7.org/fhir/StructureMap/tutorial-step5 -version 4.0.1 -ig ./maptutorial/step5/logical -ig ./maptutorial/step5/map -log test.txt -output ./maptutorial/step5/output.xml
```
  
## [step6](http://hl7.org/fhir/mapping-tutorial.html#step6) - ok

```
java -jar validator_cli.jar ./maptutorial/step6/source/source6b.xml -transform http://hl7.org/fhir/StructureMap/tutorial6d -version 4.0.1 -ig ./maptutorial/step6/logical -ig ./maptutorial/step6/map -log test.txt -output ./maptutorial/step6/output.xml
```

## [step7](http://hl7.org/fhir/mapping-tutorial.html#step7) - ok

```
./maptutorial/step7/source/source7.xml -transform http://hl7.org/fhir/StructureMap/tutorial7b -version 4.0.1 -ig ./maptutorial/step7/logical -ig ./maptutorial/step7/map -log test.txt -output ./maptutorial/step7/output.xml
```
 
## [step8](http://hl7.org/fhir/mapping-tutorial.html#step8) - ok
  
conceptmap can be embedded in map: 

```
map "http://hl7.org/fhir/StructureMap/tutorial8" = "tutorial"

conceptmap "tutorialmap" {
  prefix s = "http://hl7.org/fhir/tutorial8/codeleft"
  prefix t = "http://hl7.org/fhir/tutorial8/coderight"

  s:vonhier == t:"nach-da"
  s:test == t:test
}

uses "http://hl7.org/fhir/StructureDefinition/tutorial-left" as source
uses "http://hl7.org/fhir/StructureDefinition/tutorial-right" as target

group tutorial(source source : TLeft, target target : TRight) {
  source.d as d -> target.d = translate(d, '#tutorialmap', 'code') "rule_d";
}

```

```
java -jar validator_cli.jar ./maptutorial/step8/source/source8.xml -transform http://hl7.org/fhir/StructureMap/tutorial8 -version 4.0.1 -ig ./maptutorial/step8/logical -ig ./maptutorial/step8/map -log test.txt -output ./maptutorial/step8/output.xml
```

##[step9](http://hl7.org/fhir/mapping-tutorial.html#step9) - fails

changed rule to

```
  src.i as i where src.m < 2 -> tgt.j = i;
  src.i as i where src.m >= 2 -> tgt.k = i;
```

```
java -jar validator_cli.jar ./maptutorial/step9/source/source9b.xml -transform http://hl7.org/fhir/StructureMap/tutorial9 -version 4.0.1 -ig ./maptutorial/step9/logical -ig ./maptutorial/step9/map -log test.txt -output ./maptutorial/step9/output.xml
```
