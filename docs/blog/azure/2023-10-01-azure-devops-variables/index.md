# Azure DevOps Pipeline Variable Types - Compile-time, Queue-time, and Run-time


# **Azure DevOps: Understanding Compile-time, Queue-time, and Run-time Variables**

Azure DevOps stands as a formidable platform for continuous integration and continuous delivery (CI/CD). Its power is enriched by its versatile variable management system. This article will guide you through the maze of Azure DevOps variables, revealing the hidden quirks and behaviors across different scopes.


## **Usecases This Post Will Answer:**

Ever been puzzled about how Azure DevOps Pipelines handle variables? Wondering how to leverage your pipeline with variable scopes?

- How can I trigger my pipeline template in for loop for each of targets from list?
- How do I disable certain targets from the Azure DevOps UI?
- Is it possible to use variable groups, or other centralised means, for rendering templates?
- How to mix compile-time, queue-time, and run-time variables and/or parameters?

## **Understanding Variable Types**

Azure DevOps categorizes variables into three principal types:

1. **Compile-time Variables:** Think of these as the constants in your code. Evaluated at pipeline compilation, they remain unaltered during the pipeline's lifespan.
2. **Queue-time Variables:** Set when the pipeline is queued and predominantly seen in the Azure DevOps UI. They come to the rescue when you need to override or supply values post compilation.
3. **Run-time Variables:** The chameleons of the lot, they're dynamic and spring to life during the pipeline's execution.

## **Code Snippets, Best Practices & Insights**

Coupling insights from provided code snippets and Azure DevOps' documentation, let's decode the art of using variables:

### 1. **Templates and Compile-time**

- Templates thrive on compile-time parameters and static variables.
- Beware: passing dynamic run-time or UI-driven queue-time variables as compile-time parameters to nested templates is a no-go.

```yaml
variables:
  - name: variableHardcoded
    value: compileTime
```

#### 1.1. **Sourcing Files from other repos**

- External files in Azure DevOps are your compile-time companions. Ideal for shared configurations or templates from single centralized place.

```yaml
resources:
  repositories:
    - repository: templates
      type: git
      name: 'MyProject/infrastructure-azure'
      ref: refs/heads/dev

variables:
  - template: azure-pipelines/templates/vars.yml@templates # ! load at compile time
```

where `azure-pipelines/templates/vars.yml`

```yaml
variables:
  - name: targetsDelimiter
    value: ';'
  - name: targetsList
    value: westeurope;southuk
  ## helper variables
  - name: targetsListUpper
    value: ${{ upper(variables.targetsList) }}
  - name: targetsListLower
    value: ${{ lower(variables.targetsList) }}
```

### 2. **Mixing Variable Types**

- For conditions to work their magic, they need either queue-time or run-time variables.
- Using compile-time variables in conditions? Ensure they're declared as variables upfront.

```yaml
stages:
  - template: templates/build-stage.yml
  - ${{ each target in split(variables.targetsList, variables.targetsListDelimiter) }}:
    - template: templates/deploy-stage.yml
      parameters:
        target: ${{ target }}
```

#### 2.1. **Conditions**

where file: templates/deploy-stage.yml

```yaml
parameters:
- name: target
  displayName: AZURE TARGET REGION
  type: string
  default: ''

stages:
- stage: ShowVars${{ upper(parameters.target) }}
  dependsOn:
  variables:
    - name: target
      value: ${{ parameters.target }}
  condition: not(contains(variables.targetListDisabled, variables.target))
```

### 3. **Azure DevOps Library Group Variables**

- These are your run-time variable superstars. They dynamically enter the scene during the pipeline's runtime.

```yaml
variables:
  - group: infrastructure_dev
```

### 4. **Least precedence of UI variables**

- Variables in Azure DevOps UI are your queue-time warriors.
- However, any YAML or template-set variable can usurp the UI set variable. This safety net ensures accidental UI overrides don't wreak havoc. Rule of thumb: Set them in one place for clarity.

```yaml
# - name: targetsListDisabled # ! has to be set in Azure DevOps Pipeline UI as yml take precedence
```

## **Wrapping Up**

In the realm of Azure DevOps, mastering variables is akin to holding the keys to a treasure trove. By discerning between compile-time, queue-time, and run-time variables, you pave the way for streamlined and potent pipelines. As you journey forth, remember to always be mindful of your variable settings to guarantee a seamless CI/CD process.


---

> Author: [Robert Grzelka](https://robertgrzelka.pl)
> URL: https://robertgrzelka.pl/blog/azure/2023-10-01-azure-devops-variables/

