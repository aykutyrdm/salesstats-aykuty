# Sales Statistics

This application is implemented as a single standalone microservice that calculates real-time statistics of item sales on the eCG marketplace platform.

## Build & Run
---

### Prerequesties

- Docker engine

### Build an application image 
`docker build -t stats-app:1.0 .`

### Run and publish application image
`docker run -it --name stats-app --rm -p 8080:8080 stats-app:1.0`

## Rest API
---

### Port
- 8080

### Context path
- N/A

### GET Statistics
```
GET /statistics HTTP/1.1
Host: localhost:8080
```

### POST Sales
```
POST /sales HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

sales_amount=10.00
```

## Models
---
### `Statistic`

- Used to return last minute statistics
```
{
    String totalSalesAmount;
    String averageAmountPerOrder;
}
```
### `SubStatistic`

- Used to group statistics according to second values of sale request time
```
{
    Double totalAmount;
    Integer orderCount;
    Calendar lastUpdate;
}
```

## Memory Requirements
---

- *250.000 items are sold each minute*

- *not allowed to use persistent storage due to internal INFOSEC policies*

- *SiteOps team allocated limited cloud resources*

## Memory Solutions 

---
> **`ConcurrentLinkedQueue<SubStatisticDto> subStatisticDtos`**

- Used to keep grouped statistics with a queue in statistic service

- Size can be in the range of (0-60)

- Updated every second with expire statistic task scheduler

- Preferred due to thread safe requirement

## Performance Requirements
---
- *250.000 items are sold each minute*

- *use your CPU wisely while calculating the statistics with low time complexity*

## Performance Solutions
---

> **`@Async POST /sales`**

- Saving sale method implemented asynchronously to not block POST sale operation with statistic calculations, updates or transactions.

> **`SubStatisticDto finalSubStatisticDto`**

- In statistic service, it is used to keep sum of grouped statistics kept on queue

- Updated every second with expire statistic task scheduler

- Preferred to be able to get statistics with O(1) time complexity

> **`ExpireStatisticsTask`**

- Used to update `subStatisticDtos` queue and `finalSubStatisticDto` every second with an expire statistics thread. 

- Preferred to be able to get statistics with O(1) time complexity

## Testing
---

- To run unit tests, **build an application image** step can be applied.

**Expected Log Info**

```
[INFO] Results:
[INFO] 
[INFO] Tests run: 14, Failures: 0, Errors: 0, Skipped: 0
```


## Statistic Methods & Related Unit Tests

---

*Implemented statistic unit testing methods can be found under test folder.*


> `exipire(Calendar calendar)`

- `shouldNotUpdateFinalSubStatisticDtoWhenNotExpired()`

- `shouldUpdateFinalSubStatisticDtoWhenExpired()`

> `get()`

- `shouldGet()`


> `updateWhenExpired(SubStatisticDto subStatisticDto)`

- `shouldNotUpdateFinalSubStatisticDtoWhenNotExpired()`

- `shouldUpdateFinalSubStatisticDtoWhenExpired()`



> `updateWhenSold(double amount, Calendar calendar)`


- `shouldUpdateQueueWhenSoldIfNotMatch()`

- `shouldUpdateFinalSubStatisticDtoWhenSoldIfNotMatch()`

- `shouldExpireFinalSubStatisticDtoWhenSoldIfMatchAndExpire()`

- `shouldNotExpireFinalSubStatisticDtoWhenSoldIfMatchAndNotExpire()`

- `shouldUpdateFinalSubStatisticDtoWhenSoldIfMatchAndExpire()`

- `shouldUpdateFinalSubStatisticDtoWhenSoldIfMatchAndNotExpire()`


## Sub Statistic Methods & Unit Testing

---

*Implemented sub statistic unit testing methods can be found under test folder.*

> `create(double amount, Calendar lastUpdate)`

- `shouldCreate()`

> `isExpired(SubStatisticDto subStatisticDto, Calendar calendar)`

- `shouldExpire()`

- `shouldNotExpire()`

> `update(SubStatisticDto subStatisticDto, double amount, int orderCount, Calendar lastUpdate)`

- `shouldUpdate()`