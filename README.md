# AzureStreamAnalyticsExamples
Azure Stream Analytics Query example with Real scenario 

## 1. FTOD (First ticket of the minute)

`
SELECT 
DeviceSerialNumber,MessageTimestamp,PlantId,TruckId,ProjectId,ProjectName,
CustomerId,CustomerName,TicketNumber,TicketDateTimeinUTC,TruckSerialNumber,
TruckMake,PlantName,PlantMakeSerialNumber,Timezone,'FTOD' as alertType
INTO
[alertOutput]
FROM
[ticketInput]
where ISFIRST(mi, 2)=1
`

## 2. FTOP (First ticket of the Plant)

`
SELECT 
DeviceSerialNumber,MessageTimestamp,PlantId,TruckId,ProjectId,ProjectName,
CustomerId,CustomerName,TicketNumber,TicketDateTimeinUTC,TruckSerialNumber,
TruckMake,PlantName,PlantMakeSerialNumber,Timezone,'FTOP' as alertType
INTO
[ftopOutput]
FROM
[ticketInput]
where ISFIRST(mi, 2) OVER (PARTITION BY PlantId) = 1
`

## 3. Device Status Reporting (Online/Offline)

`
SELECT
    *,'Device Online Alert' as alertType
INTO
    [alertOutput]
FROM
    [tsfInput] 
    WHERE ISFIRST(mi, 5) OVER (PARTITION BY header.serialNumber,header.make) = 1

SELECT t1.*,'Device Offline Alert' as alertType 
INTO 
     [alertOutput2] 
FROM [tsfInput] t1 
     LEFT OUTER JOIN [tsfInput] t2 
ON t1.header.serialNumber=t2.header.serialNumber AND t1.header.make=t2.header.make 
AND DATEDIFF(minute, t1, t2) BETWEEN 1 and 5 
WHERE t2.header IS NULL
`

## 4. Milestone

# Reference Data

`
{
	"typeName": "MILESTONE",
	"serialNumber": "HPE641991",
	"description": "Milestone Rule",
	"milestoneRule": {
		"parameterToAsses": "TotalHours",
		"parameterValue": 300,
		"operation": "GREATER_THAN"
	}
}
`

`
SELECT
     t1.*,'Milestone Alert' as alertType
INTO
     [Output]
FROM
     [tsfInput] t1
     INNER JOIN [ruleInput] r1
ON t1.header.serialNumber = r1.serialNumber
AND t1.masterHourMeter.serviceMeterReading > r1.milestoneRule.parameterValue
`
