---
Title: Decision tables
--- 

# Decision tables
Decision tables are used to manage business decisions within process workflows. They adhere to the Decision Model and Notation (DMN) standard. 

Decision tables take at least one input and have at least one output. The inputs are evaluated against a set of rules defined by the modeler and then produce the relevant output(s) that match those rules to the process. 

## Using decision tables
Decision tables can be selected from the palette when designing a process. Once they have been dragged into the process definition, a dropdown list of decision tables available to the current project is displayed. 

Decision tables are handled as [service tasks](../modeling-connectors/README.md) by Activiti Enterprise and will always have the `implementation` value of `dmn-connector.EXECTUTE_TABLE`. The `name` of the decision table that is associated to the service task is stored in the [`<process-name>-extensions.json` file](../modeling-projects.md#files) as the `value` under the input `_activiti_dmn_table_`.

The following is an example of the XML for a decision table within a process definition:

```xml
<bpmn2:serviceTask id="ServiceTask_0gzhx4b" implementation="dmn-connector.EXECUTE_TABLE" />
```

The following is an example of the extensions JSON of a process containing a decision table:

```json
"mappings": {
   "ServiceTask_0v4ptl3": {
      "inputs": {
         "_activiti_dmn_table_": {
            "type": "static_value",
            "value": "Ice_cream"
         }
       }
    }
}
```

## Designing decision tables
The following is a decision table that selects the best flavor of ice cream to eat based on which day of the week it is and what the temperature is. This example will be used to assist in explaining the different elements that make up a decision table.

![Example decision table](../images/modeling-decision.png)

### General
Decision tables have a `name` and `id`, with the `name` being used to link a decision table to a service task. 

The following is the XML for the general properties of the ice cream decision table:

```xml
<decision id="Decision_Ice_cream" name="Ice_cream">
```

### Inputs
Input variables are the fields that pass values from a process into a decision table to be evaluated. In the ice cream decision table the input variables are `dayOfWeek` and `temperature` of data types `string` and `integer` respectively. Inputs also contain a label which are `Day of the week` and `Temperature (Celsius)` in the example.

The following is the XML for input variable `dayOfWeek`:

```xml
<input id="InputClause_Ice_cream" label="Day of the week" activiti:inputVariable="dayOfWeek">
   <inputExpression id="LiteralExpression_Ice_cream" typeRef="string" />
</input>
```

![Example decision table inputs](../images/decision-input.png)

Input entries, or values are the possible input values to match against for each rule in a decision table. In the ice cream example, possible values include `Monday` and `>=25`.

**Note**: Inputs can have a `-` in a column that matches any value passed in. This appears as blank in the associated XML. 

The following is the XML for the input entry of row 1:

```xml
<inputEntry id="UnaryTests_0pwpzaz">
   <text></text>
</inputEntry>
<inputEntry id="UnaryTests_0g2rex3">
   <text>&gt;35</text>
</inputEntry>
```

Input entries use the [FEEL (Friendly Enough Expression Language)](#feel) language.

### Outputs
Outputs are the result(s) that a decision table comes to after evaluating the inputs. Output columns have a `name` and a `label`. The `name` is used to pass the output value(s) from a decision table to a process. In the ice cream decision table the output `name` is `flavor` and it is of data type `string`. 

The following is the XML for the output from the ice cream decision table

```xml
<output id="OutputClause_Ice_cream" label="Flavor" name="flavor" typeRef="string" />
```

![Example decision table output](../images/decision-output.png)

Output entries are the possible outputs for each rule in a decision table. In the ice cream example, possible values include `Triple chocolate` and `Honeycomb`. 

The following is the output entry for row 10:

```xml
<outputEntry id="LiteralExpression_1olsqqv">
   <text>"Triple chocolate"</text>
</outputEntry>
```

### Rules
Each row in a decision table is known as a rule. A rule evaluates which outputs are valid for the input(s) provided. In the ice cream flavor example, the following are some of the rules:

![Example decision table rules](../images/decision-rules.png)

* On a Monday when the temperature is below 25°c, you should eat pistachio ice cream.
* On a Wednesday when the temperature is 25°c or above, you should eat vanilla ice cream.
* On a Friday you should eat triple chocolate ice cream, irrespective of temperature. 
* On Saturdays or Sundays when the temperature is 25°c or above, you should mint chocolate ice cream.
* When the temperature is above 35°c you should eat lemon sorbet, irrespective of the day. 

**Note**: If there are multiple inputs in a single rule, decision tables use an *AND* operator between the inputs.

The XML for a rule is the combination of the input and output entries with a unique rule `id`. The following is an example for rule or row 12:

```xml
<rule id="DecisionRule_1drb7gg">
   <inputEntry id="UnaryTests_07yha3g">
      <text>"Saturday","Sunday"</text>
   </inputEntry>
   <inputEntry id="UnaryTests_00i1d80">
       <text>&gt;=25</text>
   </inputEntry>
   <outputEntry id="LiteralExpression_1i6ddhb">
        <text>"Mint chocolate"</text>
   </outputEntry>
</rule>
``` 

### Hit policies
Underneath the name of the decision table is a letter that sets the hit policy for a decision table. Hit policies are used to set how rules are evaluated when a decision table is executed. 

Using the ice cream example, the letter is `F` which is a `FIRST` hit policy. This means that whilst multiple rules can be matched, only the first one matched will be returned as the output. The rules are evaluated in the order they are defined in the decision table. 

![Example decision table hit policy](../images/decision-policy.png)

Hit policies are defined at the top level of a decision table XML: 

```xml
<decisionTable id="DecisionTable_Ice_cream" hitPolicy="FIRST">
```

Other [hit policies](#hit-policy-types) can be used.  

### Annotations
On the far right of a decision table is a column for annotations. This is just a place to store notes and is only visible to the modeler. 

![Example decision table annotation](../images/decision-annotation.png)

Annotations are contained in a `description` property of a rule in the XML: 

```xml
<rule id="DecisionRule_0vx00qh">
   <description>Treat day.</description>
...
</rule>
```

## FEEL

## Hit policy types 
Hit policies define how many rules can be matched in a decision table and which of the results are included in the output. 

The default hit policy is `UNIQUE`. 

| Hit policy | Description |
| ---------- | ----------- |
| `U`: `UNIQUE` | <ul><li> Only a single rule can be matched </li></ul> <ul><li> If more than one rule is matched the hit policy is violated </ul></li> |
| `A`: `ANY` | <ul><li> Multiple rules can be matched </ul></li> <ul><li> All matching rules must have identical entries for their output </ul></li> <ul><li> If matching rules have different output entries the hit policy is violated </ul></li> | 
| `F`: `FIRST` | <ul><li> Multiple rules can be matched </ul></li> <ul><li> Only the output of the first rule that is matched will be used </ul></li> <ul><li> Rules are evaluated in the order they are defined in the decision table </ul></li> | 
| `R`: `RULE ORDER` | <ul><li> Multiple rules can be matched </ul></li> <ul><li> All outputs are returned in the order that rules are defined in the decision table </ul></li> | 
| `C`: `COLLECT` | Multiple rules can be satisfied and multiple outputs will be generated with no ordering |
| `P` : `PRIORITY` | <ul><li> Multiple rules can be matched </ul></li> <ul><li> Only the output with the highest priority will be used </ul></li> <ul><li> Priority is calculated based on the order rules are specified in descending order </ul></li> | 
| `O` : `OUTPUT ORDER` | <ul><li> Multiple rules can be matched </ul></li> <ul><li> All outputs are returned in the order that output values are defined in the decision table