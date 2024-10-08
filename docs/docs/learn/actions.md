
# LaVague Actions

## What are LaVague Actions?

🔒 To mitigate the risk of generating and executing arbitrary code generated by an LLM, we hardcoded a restricted set of web navigation actions our Agents can take.

These actions consist of:

- Clicking a web element
- Inputting a value
- Inputting a value and pressing enter
- Selecting an item from a drop down menu

We provide our Action Engine LLM with information about these actions and how they should be used. For example:

```py
Name: setValue
Description: Focus on and set the value of an input element with a specific xpath
Arguments:
  - xpath (string)
  - value (string)
```

We also provide the LLM with examples of the full output we expect from our Action Engine LLM to call these actions by outputting YAML code:

```py
- actions:
    - action:
        # Thus, we believe this is the correct element to be interacted with:
        args:
            xpath: "/html/body/div[2]/div[2]/div[3]/span/div/div/div/div[3]/div[1]/button[2]"
            value: ""
        # Then we can click on the button
        name: "click"
```

The Action Engine LLM is able to use this information to generate an output to indicate which of these actions should be performed and on which web element it should be performed on.

The Action Engine can then calls its driver's `exec_code` method, which will parse this output and call the relevant pre-defined method:

```py
 if action_name == "click":
    self.click(args["xpath"])
elif action_name == "setValue":
    self.set_value(args["xpath"], args["value"])
elif action_name == "setValueAndEnter":
    self.set_value(args["xpath"], args["value"], True)
elif action_name == "dropdownSelect":
    self.dropdown_select(args["xpath"], args["value"])
```

If the Action is not recognized or doesn't exist, you will the following error: ```raise ValueError(f"Unknown action: {action_name}")```. This can indicate poor LLM Performance as it has not successfully generated an output in the correct form.

## How to define a custom action

If you want to test out or contribute a new action, there are three key steps to follow to do.

All of these steps should be taken in the `base.py` file of the the driver you are using.

For example, if you are using the default Selenium Driver, you should implement your action in the `lavague-integrations/drivers/lavague-drivers-selenium/lavague/drivers/selenium/base.py` file.

### Add your Action's method

First of all, you can add the method for your action.

This should be written in the appropriate code for your driver. 

For example, if we wanted to add an `ClearValue` Action, which will clear an input field, we might define a corresponding `clear_value` method:

```py
def clear_value(self, xpath: str,):
    elem = self.page.locator(f"xpath={xpath}").first
    elem.clear()
```

### Handle your action to the exec_code method

You can then add an `if` clause that will call your method if the ActionEngine indicates your action should be used.

```py
elif action_name == "clearValue":
    self.clear_value(args["xpath"])
```

### Add your action to the LLM prompt

Now we need to add the action to our drivers prompt template (the `SELENIUM_PROMPT_TEMPLATE` in the case of the `SeleniumDriver`) to be sent to our LLM, teaching it how and when to indicate to use this action.

You will need to add the action's name, a description and the arguments you expect is to receive.

For example, you might add the following:

```py
Name: clearValue
Description: Focus on and clear the text of an input element with a specific xpath
Arguments:
  - xpath (string)
```

### Test you action

You can then test your action by installing your driver from your modified local package.

```bash
pip install -e lavague-integrations/drivers/lavague-drivers-selenium
```

### Contributing actions

If you think your action could be useful for the community, please feel free to open a PR with your action code for us to review.

To learn more about how to do this, see our [contribution guidelines](../contributing/general.md).