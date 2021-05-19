This is a walkthrough document for a simple IAC demo. <br/>

Azure CLI

Steps:
 1. Login into Azure account
    `az login`
 2. Create a new file called azuredeploy.json.
 3. Write `arm!` to use IntelliSense and build the body of the template.
 4. Set default subscription details
    `az account set --subscription "[subscription name]`
 5. Set default resource group name
    `az configure --defaults group=[sandbox resource group name]`
 6. Run this command to deploy your blank template
   `templateFile="azuredeploy.json"
   today=$(date +"%d-%b-%Y")
   DeploymentName="blanktemplate-"$today

   az group deployment create \
   --name $DeploymentName \
   --template-file $templateFile`