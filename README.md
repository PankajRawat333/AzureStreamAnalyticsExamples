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

## 2. Device Status Reporting (Online/Offline)

`
SELECT
    *,'Device Online Alert' as alertType
INTO
    [alertOutput]
FROM
    [tsfInput] TIMESTAMP BY header.messageTimestamp
    WHERE ISFIRST(mi, 5) OVER (PARTITION BY header.serialNumber,header.make) = 1


SELECT
    t1.*,'Device Offline Alert' as alertType
INTO
    [alertOutput2]
FROM
    [tsfInput] t1 TIMESTAMP BY header.messageTimestamp
    LEFT OUTER JOIN [tsfInput] t2 TIMESTAMP BY header.messageTimestamp
ON
    t1.header.serialNumber=t2.header.serialNumber AND t1.header.make=t2.header.make
    AND DATEDIFF(minute, t1, t2) BETWEEN 1 and 5
WHERE t2.header IS NULL
`
