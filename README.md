# Field Confidence Threshold Optimizer Tool
Kofax Transformation defaults to 80% as the default field confidence threshold. Finding the optimum threshold for each field is a tedious task.  
This tool
* visualizes true/false positives/negatives for thresholds 0%,5%,10%,15%,...90%,95% and 100%
* enables you to find the optimum threshold for each field.  
* helps you find the optimal configuration for each locator, evaluator, formatter, validation rule, and OCR profile.

| Color | Type | Correct | Valid | Human Action | 
| ----- | ---- | - | - | - |
|   🟢 | True Positive | Correct | Valid | - |
|   🔵 | False Negative | Correct | Invalid | Press ENTER |
|   🟡 | True Negative | Incorrect | Invalid | Correct Text |
|   🔴 | False Positive | InCorrect | Valid | Lost trust in system |

Accuracy = 🟢+🔵. 
Human Effort = 🔵+🟡. 
Errors = 🟡+🔴. 

## Optimizing a Project
Unfortunately many think that the way to improve a KT project is to increase accuracy 🟢+🔵. This will not lead to success. Efforts that focus on Accuracy 🟢+🔵, ignore false positives 🔴 and don't focus on reducing human effort 🔵+🟡.  

Accuracy cannot be converted into Time Savings, Cost Savings, FTE (Fulltime equivalent) reduction, ROI or productivity.  
Human effort 🔵+🟡 can directly and accuractely be converted to Time Savings, Cost Savings, FTE (Fulltime equivalent) reduction, ROI or productivity. Therefore each project should focus on human effort 🔵+🟡.

The two goals of every project should be
1. Reduce the number of errors 🔴 that humans are currently producing.
2. Reduce human effort 🔵+🟡.  

The data comes from chart underneath. We can see that 82% is the lowest threshold that still gives 0% false positives 🔴.
| Confidence | TP | FN | TN | FP | Accuracy | Human Effort |  
| - | - | - | - | - | - | - |
| | 🟢 | 🔵 | 🟡 | 🔴 | 🟢+ 🔵 | 🔵+🟡|
| 82% | 10.5% | 79.9% | 9.6% | 0.0% | 90.4% | 89.5% |
| 80% | 17.4% | 72.9% | 9.4% | 0.2% | 90.4% | 82.3% |

Reducing the threshold by 2% to 80% gives a false positive rate 🔴 of 0.2% (1 in 500) and a reduction in fields requiring human effort by 7.2 percentage points. It is always a game of compromise.  

![image](https://user-images.githubusercontent.com/103566874/163570681-910ddfe7-4c0f-4e4c-90b6-fc57cfc1f0fb.png)

The following data shows that Accuracy metrics provide no value for productivity of a solution, nor do they help in anyway with optimizing the solution.  Notice that an accuracy of about 90% can have a  productivity gain between **1.4 and 2.8**, and clearly 2.8 is twice as good at 1.4!! 2.8 means that 1.8 people can be reassigned to other work - if an employee costs $100,000 /year, then this solution will save $180,000 / year. The next step is to see how the **productivity gain** can be improved even further.  
This is an example of how the **Field Confidence Threshold Optimizer Tool** can be used.

*This example uses **1 second** as the time to confirm a correct and invalid field 🔵, and **1.5 seconds** to enter the correct value of a incorrect field 🟡.*

| Config | Confidence | TP | FN | TN | FP | Accuracy | Human Effort | Correction Ratio | Productivity gain | 
| - | - | - | - | - | - | - | - | - | - |
| | | 🟢 | 🔵 | 🟡 | 🔴 | 🟢+ 🔵 | 🔵+🟡| 🟡/(🔵+🟡) | 🔵x 1s+ 🟡x 1.5s |
| Parascript Hand Alphanumeric |82% | 0.2% | 83.0% | 16.8% | 0.0% | 83.2% | 99.8% | 17% | 1.4 |
|Parascript Hand Alpha |	80.0%|	18.3%|	72.5%|	8.9%|	0.2%|		90.8%|	81.4%|	11%|	1.7|
Parascript FirstName|79.0%|15.2%|72.3%|12.5%|0.0%|87.5%|84.8%|15%|1.6
Parascript LastName|80.0%|17.4%|72.9%|9.4%|0.2%|90.4%|82.3%| 11%|1.7
Parascript Alpha + Truth Dictionary|73.0%|60.2%|35.8%|4.0%|0.0%|96.0%|39.8%| 10%|3.6
Parascript Alpha + 2300 Names|67.0%|51.9%|37.4%|10.7%|0.0%|89.3%|48.1%| 22%|2.8
Parascript Last + 2300 Names|67.0%|51.9%|38.0%|10.1%|0.0%|89.9%|48.1%| 21%|2.8

# Step by Step Guide.
1. Create Golden Files.
2. Configure your project.
3. Test your Golden Files
4. Copy Data into Excel

## Create Golden Files

## Configure your Project

## Test your Golden Files
Add the following script to the  Project level script.
You will also need to add the functions [String_LevenshteinDistance](https://github.com/KofaxTransformation/KTScripts/blob/a08c90037fc4d0cc200722557c3ecaab27a2ab4a/FuzzyMatch.vb#L24) and [FileName_WithoutPath](https://github.com/KofaxTransformation/KTScripts/blob/a08c90037fc4d0cc200722557c3ecaab27a2ab4a/File%20System.vb#L68)
```vb
Private Sub Document_AfterProcess(ByVal pXDoc As CASCADELib.CscXDocument)
   Dim F As Long, Field As CscXDocField, TruthDoc As New CscXDocument, Truth As CscXDocField
   TruthDoc.Load(pXDoc.FileName)
   Open "c:\temp\parascript_alpha.txt" For Append As #1
   For F=0 To pXDoc.Fields.Count-1
      Set Field=pXDoc.Fields(F)
      If Field.PageIndex>-1 And TruthDoc.Fields.Exists(Field.Name) Then
         Set Truth=TruthDoc.Fields.ItemByName(Field.Name)
         If Truth.Text<>"" Then
            Print #1, FileName_WithoutPath(pXDoc.FileName)   & vbTab & vbTab & Field.Name & vbTab & Truth.Text & vbTab & Field.Text;
            Print #1, vbTab;
            Print #1, Format(Field.Confidence,"0.00%") & vbTab & Format(String_LevenshteinDistance(Field.Text,Truth.Text,IgnoreCase:=True))
         End If
      End If
   Next
   Close #1
End Sub
```
## Copy data to Excel





