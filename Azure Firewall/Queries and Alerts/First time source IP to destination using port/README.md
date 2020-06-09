## First time source IP to destination using port

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Network-Security%2Fmaster%2FAzure%2520Firewall%2FQueries%2520and%2520Alerts%2FFirst%2520time%2520source%2520ip%2520to%2520destination%2FFirstTimeSrcIpToDst.json)

This alert searches for the first time a source IP communicates with a destination using a specific port based on learing period activity.
Configurable Parameters:
- Learning period time - learning period for threashold calculation in days. Default set to 7.

```
let LearningPeriod = 7d;
let RunTime = 1h;
let StartLearningPeriod = LearningPeriod + RunTime;
let EndRunTime = RunTime - 1d;
let TrafficLogs = (AzureDiagnostics
| where OperationName == "AzureFirewallApplicationRuleLog" or OperationName == "AzureFirewallNetworkRuleLog"
| parse msg_s with * "from " srcip ":" srcport " to " dsturl ":" dstport "." *
| where isnotempty(dsturl) and isnotempty(srcip));
let LearningSrcIpToDstIpPort = (TrafficLogs
| where TimeGenerated between (ago(StartLearningPeriod) .. ago(RunTime))
| summarize LearningSrcToDsts = make_set(dsturl,10000) by srcip, dstport);
let AlertTimeSrcIpToDstIpPort = (TrafficLogs
| where TimeGenerated between (ago(RunTime) .. ago(EndRunTime))
| extend AlertTimeDst = dsturl
| distinct AlertTimeDst ,srcip, dstport);
AlertTimeSrcIpToDstIpPort
| join kind=leftouter (LearningSrcIpToDstIpPort) on srcip, dstport
| mv-expand LearningSrcToDsts
| where AlertTimeDst != LearningSrcToDsts
| summarize LearningSrcToDsts = make_set(LearningSrcToDsts,10000) by srcip, AlertTimeDst
```

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.