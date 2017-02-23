---
external help file: Test-Expression-help.xml
schema: 2.0.0
---

# Test-Expression

## SYNOPSIS
Test a PowerShell expression.

## SYNTAX

### Interval (Default)
```
Test-Expression [-Expression] <ScriptBlock> [-ArgumentList <Object[]>] [-Count <Int32>] [-Interval <Double>]
 [-IncludeExpression] [-AsJob] [<CommonParameters>]
```

### Random
```
Test-Expression [-Expression] <ScriptBlock> [-ArgumentList <Object[]>] [-Count <Int32>] -RandomMinimum <Double>
 -RandomMaximum <Double> [-IncludeExpression] [-AsJob] [<CommonParameters>]
```

## DESCRIPTION
This command will test a PowerShell expression or scriptblock for a specified number of times and calculate the average runtime, in milliseconds, over all the tests. 
The output will also show the median and trimmed values.

The median is calculated by sorting the values in ascending order and selecting the value in the center of the array.
If the array has an even number of elements then the median is the average of the two values in the center.
The trimmed value will toss out the lowest and highest values and average the remaining values.
This may be the most accurate indication as it will eliminate any small values which might come from caching and any large values which may come a temporary shortage of resources.
You will only get a value if you run more than 1 test.

## EXAMPLES

### -------------------------- EXAMPLE 1 --------------------------
```
PS C:\> $cred = Get-credential globomantics\administrator
PS C:\> $c = "chi-dc01","chi-dc04"
PS C:\> Test-Expression {param ([string[]]$computer,$cred) get-wmiobject -class win32_logicaldisk -computername $computer -credential $cred } -argumentList $c,$cred


Tests        : 1
TestInterval : 0.5
AverageMS    : 1990.6779
MinimumMS    : 1990.6779
MaximumMS    : 1990.6779
MedianMS     : 1990.6779
TrimmedMS    :
PSVersion    : 5.1.14409.1005
OS           : Microsoft Windows 8.1 Enterprise
```

Test a command once passing an argument to the scriptblock. There is no TrimmedMS value because there was only one test.

### -------------------------- EXAMPLE 2 --------------------------
```
PS C:\> $sb = {1..1000 | foreach {$_*2}}
PS C:\> test-expression $sb -count 10 -interval 2

Tests        : 10
TestInterval : 2
AverageMS    : 72.78199
MinimumMS    : 29.4449
MaximumMS    : 110.6553
MedianMS     : 90.3509
TrimmedMS    : 73.4649625
PSVersion    : 5.1.14409.1005
OS           : Microsoft Windows 8.1 Enterprise


PS C:\> $sb2 = { foreach ($i in (1..1000)) {$_*2}}
PS C:\> test-expression $sb2 -Count 10 -interval 2

Tests        : 10
TestInterval : 2
AverageMS    : 6.40283
MinimumMS    : 0.7466
MaximumMS    : 22.968
MedianMS     : 2.781
TrimmedMS    : 5.0392125
PSVersion    : 5.1.14409.1005
OS           : Microsoft Windows 8.1 Enterprise
```

These examples are testing two different approaches that yield the same results over a span of 10 test runs, pausing for 2 seconds between each test.
The values for Average, Minimum and Maximum are in milliseconds.

### -------------------------- EXAMPLE 3 --------------------------
```
PS C:\> Test-expression {Param([string]$computer) get-service bits,wuauserv,winrm -computername $computer} -count 5 -IncludeExpression -argumentList chi-hvr2


Tests        : 5
TestInterval : 500
AverageMS    : 15.53376
MinimumMS    : 11.6745
MaximumMS    : 24.9331
MedianMS     : 13.8928
TrimmedMS    : 13.6870666666667
PSVersion    : 5.1.14409.1005
OS           : Microsoft Windows 8.1 Enterprise
Expression   : Param([string]$computer) get-service bits,wuauserv,winrm -computername $computer
Arguments    : {chi-hvr2}
```

Include the tested expression in the output.

### -------------------------- EXAMPLE 4 --------------------------
```
PS C:\> Test-Expression { get-eventlog -list } -count 10 -Interval 5 -asjob


Id     Name            PSJobTypeName   State         HasMoreData     Location             Command                  
--     ----            -------------   -----         -----------     --------             -------                  
184    Job184          RemoteJob       Running       True            WIN81-ENT-01         ...

PS C:\> receive-job 184 -keep

Tests        : 10
TestInterval : 5
AverageMS    : 2.80256
MinimumMS    : 0.7967
MaximumMS    : 14.911
MedianMS     : 1.4469
TrimmedMS    : 1.5397375
PSVersion    : 5.1.14409.1005
OS           : Microsoft Windows 8.1 Enterprise
RunspaceId   : f30eb879-fe8f-4ad0-8d70-d4c8b6b4eccc
```
Run the test as a background job. When the job is complete, get the results.

### -------------------------- EXAMPLE 5 --------------------------
```
PS C:\> {1..1000} | Test-Expression -count 10 -randomminimum 1 -randommaximum 10

Tests        : 10
TestInterval : Random
AverageMS    : 0.63899
MinimumMS    : 0.2253
MaximumMS    : 3.9062
MedianMS     : 0.24475
TrimmedMS    : 0.2823
PSVersion    : 5.1.14409.1005
OS           : Microsoft Windows 8.1 Enterprise
```
Pipe a scriptblock to be tested.

## PARAMETERS

### -Expression
The scriptblock you want to test.


```yaml
Type: ScriptBlock
Parameter Sets: (All)
Aliases: sb

Required: True
Position: 1
Default value: None
Accept pipeline input: True (ByValue)
Accept wildcard characters: False
```

### -ArgumentList
An array of parameters to pass to the test scriptblock.
Arguments are positional. If passing an array for a value enter with @().


```yaml
Type: Object[]
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -Count
The number of times to test the scriptblock.



```yaml
Type: Int32
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: 1
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -Interval
How much time to sleep in seconds between each test.
Maximum is 60.
You may want to use a sleep interval to mitigate possible caching effects.



```yaml
Type: Double
Parameter Sets: Interval
Aliases: sleep

Required: False
Position: Named
Default value: 0.5
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -IncludeExpression
Include the test scriptblock in the output.


```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: ie

Required: False
Position: Named
Default value: False
Accept pipeline input: True (ByPropertyName)
Accept wildcard characters: False
```

### -AsJob
Run the tests as a background job.



```yaml
Type: SwitchParameter
Parameter Sets: (All)
Aliases: 

Required: False
Position: Named
Default value: False
Accept pipeline input: False
Accept wildcard characters: False
```

### -RandomMinimum
You can also specify a random interval by providing a random minimum and maximum values in seconds.



```yaml
Type: Double
Parameter Sets: Random
Aliases: min

Required: True
Position: Named
Default value: 0
Accept pipeline input: False
Accept wildcard characters: False
```

### -RandomMaximum
You can also specify a random interval by providing a random minimum and maximum values in seconds.



```yaml
Type: Double
Parameter Sets: Random
Aliases: max

Required: True
Position: Named
Default value: 0
Accept pipeline input: False
Accept wildcard characters: False
```

### CommonParameters
This cmdlet supports the common parameters: -Debug, -ErrorAction, -ErrorVariable, -InformationAction, -InformationVariable, -OutVariable, -OutBuffer, -PipelineVariable, -Verbose, -WarningAction, and -WarningVariable. For more information, see about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).

## INPUTS

### scriptblock

## OUTPUTS

### Custom measurement object

## NOTES
Learn more about PowerShell: http://jdhitsolutions.com/blog/essential-powershell-resources/

## RELATED LINKS
[Online Version:](https://github.com/jdhitsolutions/Test-Expression/blob/master/docs/Test-Expression.md)

[Measure-Command](http://go.microsoft.com/fwlink/?LinkId=821828)





