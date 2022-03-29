---
title: "Insert Large Data Manually To Avoid Timeout"
date: 2021-08-10T08:06:25+06:00
description: "ways to prevent jta timeout when insert/update large data into database"
menu:
  sidebar:
    name: Insert Large Data
    identifier: Insert-Large-Data
    parent : SQL-directory
    weight: 10
<!--- hero: images/data.jpg --->
---
As part of a batch project, I have to insert/update large data (up to millions rows) into a MSSQL database. This requires insert into a hundred rows in one statement, then looping through all datas, such long transaction may encounter an error of `jta transaction unexpectedly rolled back (maybe due to a timeout).` Here's how to handle this exception.

## Basic setting 
The transaction annotation timeout value, can be set up to Integer.MAX_VALUE. However if that doesn't work for big data insert, try take the annotation out, don't let Spring be in charge of the lifecyle of the transaction, let's handle the methods `begin()`, `close()`, `commit()` of the transaction manually by ourselves. 
```
@Transactional(rollbackFor = Exception.class, timeout = 60)
```  

Setting the number of rows per commit in jdbc properties is also a way to prevent transaction timeout. The example below determines 100 inserts or updates to be carried out in a single database hit.
```
hibernate.jdbc.batch_size=100
```
Below I'll show how to manually determines the number of inserts that are sent to the database at one time for execution.  

## Commit once Every 100 entities
```java
    @PersistenceUnit(unitName="YourEntityManagerFactoryBean")
    private EntityManagerFactory entityManagerFactory;

    public void batchSaveAll(Set<Entity> entitySet) {
        int i=0;
        EntityManager em = entityManagerFactory.createEntityManager();
        em.getTransaction().begin();
        Iterator<Entity> iterator = entitySet.iterator();
        while (iterator.hasNext()){
            em.persist(iterator.next());
            if(++i%100==0){
                em.flush();
                em.clear();
                em.getTransaction().commit();
                em.getTransaction().begin();
            }
            iterator.remove();
        }
        em.flush();
        em.clear();
        em.getTransaction().commit();
        em.close();
    }
```

## JUnit Test
```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes= Config.class)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@DataJpaTest
public class Test {

    @Autowired
    Repository repository;

    @Test
    public void testSaveAll(){
        Set<Entity> set = new HashSet<>(1000);
        for(int i=0;i<1000;i++){
            set.add(getEntity());
        }
        repository.batchSaveAll(set);
        List<Entity> list2 = repository.findAll();
        assertEquals(1000,list2.size());
    }

    /** Generate Entity */
    private Entity getEntity(){
        Entity entity = new Entity();
        entity.setId(RandomStringUtils.randomAlphabetic(12));
        entity.setMark1(RandomStringUtils.randomAlphabetic(1));
        entity.setMark2(RandomStringUtils.randomAlphabetic(1));
        entity.setReceiptNo(RandomStringUtils.randomAlphabetic(10));
        return entity;
    }
```

## Insert to a temp table
If inserting large data into temp table is your case, without persist() method, which requires an instance of Entity, Spring JPA also allows us to use `createNativeQuery()` `executeUpdate()` for custom sql statement, then we can set up the start and close for a transaction in order to commit every 100 rows. In case of no entity, I put the column values inside a List.
```java
   public void insertTempTable(Temptable bean, List<Map<String, String>> records) throws Exception{
        String tableName = StringUtils.stripToEmpty(bean.getTableName());
        if(CollectionUtils.isNotEmpty(records)){
            int size = records.size();
            LOGGER.debug("*** Total records inserting to temp table {} is {}", tableName, size);

            EntityManager em = entityManagerFactory.createEntityManager();
            em.getTransaction().begin();

            StringBuilder insertSql = new StringBuilder();
            insertSql.append("INSERT INTO "+ tableName +" (receiveNo,exitDate,inspectTime,exitPort,exitFromTo) VALUES (?1,?2,?3,?4,?5)");

            for(int i=0; i<size; i++) {
                String receiveNo = records.get(i).get("FirstReceiveNo");
                String exitDate = records.get(i).get("ExitDate");
                String inspectTime = records.get(i).get("ExitInspectTime");
                String exitPort = records.get(i).get("ExitPort");
                String exitFromTo = records.get(i).get("ExitFromTo");
                em.createNativeQuery(insertSql.toString())
                        .setParameter(1, receiveNo)
                        .setParameter(2, exitDate)
                        .setParameter(3, exitInspectTime)
                        .setParameter(4, exitPort)
                        .setParameter(5, exitFromTo)
                        .executeUpdate();
                //commit every 100 records to avoid transaction time out
                if(i!=0 && i%100==0){
                    em.flush();
                    em.clear();
                    em.getTransaction().commit();
                    em.getTransaction().begin();

                    //reset the stringBuilder
                    insertSql.setLength(0);
                    insertSql.append("INSERT INTO "+ tableName +" (receiveNo,exitDate,inspectTime,exitPort,exitFromTo) VALUES (?1,?2,?3,?4,?5)");
                    }
            }
            //The rest of the data (not enough of 100 rows) should not be forgotten
            em.flush();
            em.clear();
            em.getTransaction().commit();
            em.close();
        }else{
            return;
        }
    }
``` 