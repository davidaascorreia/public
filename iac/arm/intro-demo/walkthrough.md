This is a walkthrough document for a simple IAC demo. <br/>

Azure CLI

Steps:
 1. Login into Azure account
    `az login`
 2. Create a new file called azuredeploy.json.
 3. Write `arm!` to use IntelliSense and build the body of the template. Save file / merge to master.
 4. Set default subscription details
    `az account set --subscription "[subscription name]`
 5. Set default resource group name
    `az configure --defaults group=[sandbox resource group name]`
 6. Run this command to deploy your blank template <br>
   `templateFile="azuredeploy.json"` <br>
   `today=$(date +"%d-%b-%Y")` <br>
   `DeploymentName="blanktemplate-"$today` <br>
   `az deployment group create --name $DeploymentName --template-file $templateFile`
 7. Add a storage resource to the template. Select `arm-storage`.
 8. Deploy the updated ARM template <br>
   `templateFile="azuredeploy.json"` <br>
   `today=$(date +"%d-%b-%Y")` <br>
   `DeploymentName="addstorage-"$today` <br>
   `az deployment group create --name $DeploymentName --template-file $templateFile`
 9. Add a parameter into the parameter section in *azuredeploy.json*. Name the parameter *storageName*.
 10. Add a minLength value of 3 and a maxLength value of 24. Add a description value of The name of the Azure storage resource.
 11. Save file / merge to master.
 12. Deploy <br>
   `templateFile="azuredeploy.json"` <br>
   `today=$(date +"%d-%b-%Y")` <br>
   `DeploymentName="addnameparameter-"$today` <br>
   `az deployment group create --name $DeploymentName --template-file $templateFile --parameters storageName=<your unique name>`
 13. Add another parameter *storageSKU* to limit allowed values. Set default value "Standard_LRS" and following allowed values: Standard_LRS, Standard_GRS, Standard_RAGRS, Standard_ZRS, Premium_LRS, Premium_ZRS, Standard_GZRS, Standard_RAGZRS
 14. Add comments and description "This is the allowed values for an Azure storage account"
 15. Save file. Deploy testing both allowed and not allowed values.
