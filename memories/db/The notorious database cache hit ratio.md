# The notorious database cache hit ratio

https://medium.com/mongodb-performance-tuning/the-notorious-database-cache-hit-ratio-c7d432381229

***

## Database 의 두 가지 원칙

- disk I/O 는 여전히 느리다.
- 메모리를 더 많이 둔다는 건 I/O 를 줄인다는 의미다.

## The Cache Hit Ratio (called Buffer Cache Hit Ratio)

- Cache Hit Ratio = Cache I/O / Total I/O Request
    - 데이터 블럭의 요청수와 관련이 있다.

## Cache Miss Rate 계산식 (조금 이상함.)

```javascript
function missrate(intervalMs) {
  var statsStart = db.serverStatus().wiredTiger.cache;
  var startTime = new Date();

  sleep(intervalMs);

  var endTime = new Date();
  var statsEnd = db.serverStatus().wiredTiger.cache;

  var logicalReads =
    statsEnd['pages requested from the cache'] -
    statsStart['pages requested from the cache'];
  var physicalReads =
    statsEnd['pages read into cache'] - statsStart['pages read into cache'];
  var elapsedTime = endTime - startTime;
  var missRate = physicalReads * 100 / logicalReads;
  var logicalReadRate =
    Math.round(logicalReads * 1000 * 100 / elapsedTime) / 100;
  var physicalReadRate =
    Math.round(physicalReads * 1000 * 100 / elapsedTime) / 100;

  print('Elapsed time (ms)', elapsedTime);
  print('logical Read Rate IO/s', logicalReadRate);
  print('physical Read Rate IO/s', physicalReadRate);
  print('wiredTiger miss rate', Math.round(missRate * 100) / 100);
}
```

- `pages read into cache` 는 cache 에서 읽어온 페이지 수를 말하지 않나. (이게 맞는듯.)


## WiredTiger Cache Size 를 변경할려면

```javascript
db.getSiblingDB("admin").runCommand({ 
     setParameter: 1, 
     wiredTigerEngineRuntimeConfig: "cache_size=500M" 
});
```

## 캐시를 늘려도 효과가 없는 경우

- 이미 모든 데이터가 캐시에 있다면.

- 캐시에 있는 데이터를 두 번다시 읽을 일이 없다면.

- (application 의 workingSet 계산을 잘해야한다.) 
