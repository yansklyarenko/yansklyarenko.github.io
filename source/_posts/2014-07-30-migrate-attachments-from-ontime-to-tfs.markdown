---
layout: post
title: "Migrate attachments from OnTime to TFS"
date: 2014-07-31 23:32
comments: true
categories: ontime tfs migrate attachment
---

When you move from one bug tracking system to another, the accuracy of the process is very important. A single missing point can make a work item useless. An attached image is often worth a thousand words. Hence, today's post is about migrating attachments from [OnTime][1] to [TFS][2].

> NOTE: The samples in this post rely on OnTime SDK, which was replaced by a [brand new REST API][3]. 

OnTime SDK is a set of web services, and each "area" is usually covered by one or a number of web services. The operations with attachments are grouped in `/sdk/AttachmentService.asmx` web service. 

So, the first thing to do is to grab all attachments of the OnTime defect:

```c#
var rawAttachments = _attachmentService.GetAttachmentsList(securityToken, AttachmentSourceTypes.Defect, defect.DefectId);
```
This method returns a `DataSet`, and you'll have to enumerate its rows to grab the useful data:

```c#
var attachments = rawAttachments.Tables[0].AsEnumerable();
foreach (var attachment in attachments)
{
  // wi is a TFS work item object
  wi.Attachments.Add(GetAttachment(attachment));
}
```

Now, let's take a look at the `GetAttachment` method, which actually does the job. It accepts the `DataRow`, and returns the TFS `Attachment` object:

```c#
private Attachment GetAttachment(DataRow attachmentRow)
{
  var onTimeAttachment = _attachmentService.GetByAttachmentId(securityToken, (int)attachmentRow["AttachmentId"]);

  var tempFile = Path.Combine(Path.GetTempPath(), onTimeAttachment.FileName);
  if (File.Exists(tempFile))
    File.Delete(tempFile);
  File.WriteAllBytes(tempFile, onTimeAttachment.FileData);

  return new Attachment(tempFile, onTimeAttachment.Description);
}
```
Couple of things to notice here:

 - you have to call another web method to pull binary data of the attachment
 - OnTime attachment metadata is rather useful and can be moved as is to TFS, for instance, attachment description
 
Finally, when a new attachment is added to the TFS work item, "increment" the `ChangedDate` of the work item before saving it. The TFS server often refuses saving work item data in case the previous revision has exactly the same date/time stamp. Like this (always works):

```c#
wi[CoreField.ChangedDate] = wi.ChangedDate.AddSeconds(5);
wi.Save();
```
Hope it's useful. Good luck! 

  [1]: http://www.axosoft.com/
  [2]: http://en.wikipedia.org/wiki/Team_Foundation_Server
  [3]: http://developer.axosoft.com/api
