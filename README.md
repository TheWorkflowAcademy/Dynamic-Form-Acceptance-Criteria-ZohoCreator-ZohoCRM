# Dynamic-Form-Acceptance-Criteria-ZohoCreator-ZohoCRM
This setup enables a Zoho Creator form submission's acceptance/ rejection criteria to be dynamically configured in Zoho CRM instead of having them hard-coded in the Creator script.

## Core Idea
Some Creator forms have complex acceptance/ rejection criteria that may be subjected to changes from time-to-time. Having all that hard-coded into the script can be both tedious and unscalable. This workaround establishes a dynamic link between Creator and CRM allowing users to simply set the criteria they desire on a record in a CRM custom module with 0 coding knowledge needed.

## Example Case
You have a Creator Form for students to apply for online courses. In the form, you have questions with different acceptance criteria based on the **Course Type** that they choose. At the same time, you work with certain organizations partners where a different set of criteria applies if an applicant keys in a Participation Code provided by the organization (eg; certain qualification questions are renderred unnecessary). Upon submission of the form, the applicant will either be accepted/ rejected to the course based on these different individual criteria.

## Tutorial - CRM
### Create a Custom Module in CRM
Create a "Qualification Criteria" Custom Module in CRM
* All criteria will be created as records in this module.

### Create Custom Fields
In the module, create custom fields based on your criteria requirements. This should mirror your Creator form questions. Below are some examples of different field types your form may include:
* Yes/ No Questions - Picklist Field with the following values
  * Yes
  * No
  * N/A *- field is not applicable (the script will bypass this qualification criteria).*
* Multi-Select Questions (the script will check if the value(s) selected by applicants of the Creator form is/ are present on this CRM field. If yes - pass, if no - fail).
  * Value A
  * Value B
  * Value C
  * etc..
* "Applicable to Me" Questions. 
  * This allows for a more unique/ customisable criteria setup. It should be a section with a **parent field** (where criteria is set) and **child fields** (where the values are set).
  * The values in the child fields will also be programmed later to dynamically populate the Creator form.
  * **Parent** Field: "Applicable to Me Requirements" (criteria is set here). Applicants are required to select All/ Any 1/ Any 2/ Any 3/ None of the child field values to be accepted. If it is "N/A", the script will bypass this qualification criteria.
    * All
    * Any 1
    * Any 2
    * Any 3
    * None
    * N/A
  * **Child** Fields: the parent field will reference the values in the child fields below, these values will also be used to populate the Creator Form.
    * Applicable to Me 1
    * Applicable to Me 2
    * Applicable to Me 3
    * Applicable to Me 4 
    * Applicable to Me 5
    * Add more if you need..
* "Partner" field
  * A lookup field to Accounts for the Creator script to identify unique organization-specific qualification criteria.
    
### Create Records in the "Qualification Criteria" module
* Create a record for each Course Type with its unique qualifying criteria - Course Type A (General), Course Type B (General), etc. This would take care of all general submissions. 
* To account for organization-specific criteria, a separate record can be created. For example, you have a special deal with an organisation called "Umbrella Corp" for Course Type A. For applicants with Participation Codes from Umbrella Corp, there is a different set of qualifying criteria. We can do this by creating another record for Course Type A with different criteria called "Course Type A (Umbrella Corp)". Do make sure to select the organization in the "Partner" lookup field.

## Tutorial - Creator 
### Create a Workflow - "on user input of a field"
The following script needs to be written on user input of a field of the Creator form.
* Record Event (Created) > Form Event (User input of a field) > Choose Field (Course Type)

### Creator script to dynamically populate Creator form
The script below searches the Qualification Criteria record on CRM based on the Course Type input on the form, and then dynamically populates the "Applicable to Me" section as field values on the Creator Form. If there are no values at all, the entire field on the Form will be hidden.
* Important Note: To assist searching in CRM, we have also set up a "tag" system where tags will be added to the Contact record based on what the applicant had selected (all applicants regardless of acceptance will have a Contact created in CRM).
  * For this to work correctly, it is important to follow the **exact format** of adding "tag" into the "Applicable to Me" child fields as shown in the example below:
    * I will be actively looking for a remote job within the next 1-2 months. TAG: remotejob

```javascript
qualificationrecord = zoho.crm.searchRecords("Qualification_Criteria","Course_Type:equals:" + input.Course_Type);
if(qualificationrecord.size() > 0)
{
	q = qualificationrecord.get(0);
	taglist = "";
	applist = List();
	if(q.get("Applicable_to_Me_1") != null)
	{
		applist.add(q.get("Applicable_to_Me_1").getPrefix("TAG:").trim());
		taglist = taglist + "\"" + q.get("Applicable_to_Me_1").getPrefix("TAG:").trim() + "\"" + ":" + "\"" + q.get("Applicable_to_Me_1").getSuffix("TAG:").trim() + "\"" + ",";
	}
	if(q.get("Applicable_to_Me_2") != null)
	{
		applist.add(q.get("Applicable_to_Me_2").getPrefix("TAG").trim());
		taglist = taglist + "\"" + q.get("Applicable_to_Me_2").getPrefix("TAG:").trim() + "\"" + ":" + "\"" + q.get("Applicable_to_Me_2").getSuffix("TAG:").trim() + "\"" + ",";
	}
	if(q.get("Applicable_to_Me_3") != null)
	{
		applist.add(q.get("Applicable_to_Me_3").getPrefix("TAG").trim());
		taglist = taglist + "\"" + q.get("Applicable_to_Me_3").getPrefix("TAG:").trim() + "\"" + ":" + "\"" + q.get("Applicable_to_Me_3").getSuffix("TAG:").trim() + "\"" + ",";
	}
	if(q.get("Applicable_to_Me_4") != null)
	{
		applist.add(q.get("Applicable_to_Me_4").getPrefix("TAG").trim());
		taglist = taglist + "\"" + q.get("Applicable_to_Me_4").getPrefix("TAG:").trim() + "\"" + ":" + "\"" + q.get("Applicable_to_Me_4").getSuffix("TAG:").trim() + "\"" + ",";
	}
	if(q.get("Applicable_to_Me_5") != null)
	{
		applist.add(q.get("Applicable_to_Me_5").getPrefix("TAG").trim());
		taglist = taglist + "\"" + q.get("Applicable_to_Me_5").getPrefix("TAG:").trim() + "\"" + ":" + "\"" + q.get("Applicable_to_Me_5").getSuffix("TAG:").trim() + "\"" + ",";
	}
	if(applist.size() > 0)
	{
		show Applicable_To_Me;
	}
	for each  a in applist
	{
		input.Applicable_To_Me:ui.add(a);
	}
	input.TagInfo = "{" + taglist.removeLastOccurence(",") + "}";
}
else
{
	hide Applicable_To_Me;
}
```

### Create a Workflow - "on form submission"
The following script needs to be written on submission of the Creator form.
* Record Event (Created) > Form Event (Successful Form Submissions)

### Get the right record on Qualification Criteria
Based on the form input, the script will find the Qualification Criteria record based on the Course Type and check if it's a "General" criteria or a special organization-specific criteria. 

```javascript
qualificationrecords = zoho.crm.searchRecords("Qualification_Criteria","Course_Type:equals:" + input.Course_Type);
usepartnercriteria = false;
for each  r in qualificationrecords
{
	if(r.get("Partner") != null)
	{
		if(Organization_Name = ifNull(r.get("Partner").get("name"),"Nope"))
		{
			usepartnercriteria = true;
			partnercriteria = r;
		}
	}
	else
	{
		criteria = r;
	}
}
if(usepartnercriteria = true)
{
	criteria = partnercriteria;
}
```

### Set the Counters
* Before the script parses through the criteria, the default value of the "result" is set as "Accepted". At any point of time when an applicant selects the "wrong" answer, the "result" variable will be changed to "Rejected".
* "reason" and "reason_admin" are variables used to build the fail message for the reference of both the applicants (via email) and the CRM users (via Notes in the Contact record) respectively - this is optional.

```javascript
q = criteria;
applist = List();
result = "Accepted";
reason = "";
reason_admin = "";
```

### Standard Yes/ No Fields
*Note: This is an example script. Change the field name ("Are_you_above_18") accordingly.*
```javascript
if(q.get("Are_you_above_18") != "N/A")
{
	if(Are_you_above_18 != q.get("Are_you_above_18"))
	{
		result = "Rejected";
		reason = reason + "• You need to be above 18 years old to join this course." + "\n";
		reason_admin = reason_admin + "• Is not above 18 years old." + "\n";
	}
}
```

### Multi-Select field with Specific Options to Select
*Note: This is an example script. Change the field name ("Reason_for_taking_this_course") accordingly.*
```javascript
mainreason = q.get("Reason_for_taking_this_course");
if(mainreason != "N/A")
{
	if(mainreason.size() > 0 && !mainreason.contains(input.Reason_for_taking_this_course))
	{
		result = "Rejected";
		reason = reason + "• Your reason for taking this course is not what we are looking for." + "\n";
		reason_admin = reason_admin + "• Selected the wrong reason for taking the course." + "\n";
	}
}
```

### Applicable to Me section with all Present Options (Unique Qualification Multi Select)
```javascript
if(q.get("Applicable_To_Me_Requirements") != "N/A")
{
	if(q.get("Applicable_To_Me_Requirements") = "All")
	{
		iterator = {1,2,3,4,5};	// Change to the number of “child” fields you have in the “Applicable to Me” section in CRM
		n = 0;
		for each  i in iterator
		{
			fieldname = "Applicable_To_Me_" + i;
			if(q.get(fieldname) != null)
			{
				n = n + 1;
			}
		}
		if(Applicable_To_Me.size() != n)
		{
			result = "Rejected";
			reason = reason + "• You needed to select all of the 'Applicable to Me' options." + "\n";
			reason_admin = reason_admin + "• Did not select all of the 'Applicable to Me' options." + "\n";
		}
	}
	else if(q.get("Applicable_To_Me_Requirements") = "None")
	{
		if(Applicable_To_Me.size() != 0)
		{
			result = "Rejected";
			reason = reason + "• You needed to select NONE of the 'Applicable to Me' options." + "\n";
			reason_admin = reason_admin + "• Selected one of more of the 'Applicable to Me' options." + "\n";
		}
	}
	else if(q.get("Applicable_To_Me_Requirements").contains("Any"))
	{
		if(Applicable_To_Me.size() < q.get("Applicable_To_Me_Requirements").right(1).toLong())
		{
			result = "Rejected";
			reason = reason + "• You need to have chosen at least " + q.get("Applicable_To_Me_Requirements").right(1).toLong() + " of the 'Applicable to Me' options." + "\n";
			reason_admin = reason_admin + "• Did not select at least " + q.get("Applicable_To_Me_Requirements").right(1).toLong() + " of the 'Applicable to Me' options." + "\n";
		}
	}
}
```

### Set the Acceptance/ Rejection Actions
* If the applicant is Rejected:
  * Build the fail messages for the applicant and CRM user as collected by the "reason" and "reason_admin" variables.
  * Set the necessary **rejection actions**. In this example, we do the following (functions not included in the script for brevity):
    * Email the rejected applicants with the fail message.
    * Create/ Update Contact in CRM with a Note containing the fail reason and other necessary fields.
    * Redirect applicant to your website homepage.
* If the applicant is Accepted
  * Set the necessary **acceptance actions**. In this example, we do the following (functions not included in the script for brevity):
    * Create/ Update Contact in CRM with necessary fields.
    * Assign a Program Coordinator for the applicant.
    * Create an Invoice with the applicant details.
    * Redirect applicant to the Invoice payment page.

```javascript
if(result = "Rejected")
{

  //Construct the fail message
	failmessage = "Dear " + input.First_Name + " " + input.Last_Name + "," + "\n\n" + "Sorry, you were not accepted for the " + input.Course_Type + " Course. See below for the reason(s):" + "\n" + reason + "\n" + "Thank you for your application.";
	failmessage_admin = "This Contact was just Rejected from an application for " + input.Course_Type + ". Here are the reasons:" + "\n" + reason_admin;
  
  //Insert your rejection actions here
  
}
else
{

  //Insert your acceptance actions here
  
}
```
