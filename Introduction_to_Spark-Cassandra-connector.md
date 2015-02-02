Slide 18

```scala
val conf = new SparkConf(true).setAppName("basic_example").setMaster("local[3]");
val sc = new SparkContext(conf); 
```

```scala
val people = List(("jdoe","John DOE", 33),
			   ("hsue","Helen SUE", 24), 
			   ("rsmith", "Richard Smith", 33));

```

Slide 19

```scala
  // count users by age
 val counByAge = sc.parallelize(people) 
			     .map(tuple => (tuple._3, tuple))
			     .groupByKey()
			     .countByKey();	

 println("Count by age : "+countByAge);
```

Slide 20

```scala
  // count users by age
 val counByAge = sc.parallelize(people)  //split the people list into different chunks (partitions)
			     .map(tuple => (tuple._3, tuple))  //("jdoe","John DOE", 33) => (33,(("jdoe",…))
			     .groupByKey()  //{33 -> (("jdoe",…), ("rsmith",…)), 24->("hsue",…))}
			     .countByKey(); //{33 -> 2, 24->1}

 println("Count by age : "+countByAge);
 //Count by age = Map(33 -> 1, 24 -> 1)

```

Slide 21

```scala
val parallelPeople: RDD[(String, String, Int)] = sc.parallelize(people);

val extractAge: RDD[(Int, (String, String, Int))] = parallelPeople.map(tuple => (tuple._3, tuple))

val groupByAge: RDD[(Int, Iterable[(String, String, Int)])] = extractAge.groupByKey()

val countByAge: Map[Int, Long] = groupByAge.countByKey()

```

Slide 58

```scala
 case class Person(login: String, name: String, age: Int)
 case class Email(login: String, content: String, date: Date)

 val people = sc.textFile("people.txt").map(_.split(","))
			.map(p => Person(p(0),p(1), p(2).trim.toInt))

 val emails = sc.textFile("emails.txt").map(_.split(","))
			.map(e => Email(e(0),e(1), p(E).trim.toDate))

 val eavesdrop = emails.map(e => (e.login,e)).join(people.map(p => (p.login,p))
			  .filter { case (login,(e,p)) => e.date ≥ ‘2015-01-01 00:00:00’}
			  .foreach { case (login,(e,p)) => println("""User : $p.name, email: e.content""")}

 println("People by age: "+people.map(p => (p.age,1)).groupByKey().countByKey())

```

Slide 64

```scala
 case class Person(login: String, name: String, age: Int, countryCode: String)
 case class Country(code: String, name: String)

 val people = sc.textFile("people.txt").map(_.split(","))
			.map(p => Person(p(0),p(1), p(2).trim.toInt))

 val country = sc.textFile("countries.txt").map(_.split(","))
			.map(c => Country(c(0),e(1))

 val countByCountry = people.map(p => (p.countryCode, 1))
			   .join(country.map(c => (c.code, c.name))
			  .map { case (countryCode,(one,name)) => (countryCode,name)}
         			  .reduceByKey().countByKey();
```

Slide 65

```scala
 …
 val people = sc.textFile("people.txt").map(_.split(","))
			.map(p => Person(p(0),p(1), p(2).trim.toInt))

 val country = sc.textFile("countries.txt").map(_.split(","))
			.map(c => Country(c(0),e(1))

 //Broadcast shared variable
 val countryBc = sc.broadcast(country)

 val countByCountry = people.map(p => (p.countryCode, 1))
			   .join(countryBc.value.map(c => (c.code, c.name))
			   .map { case (countryCode,(one,name)) => (countryCode,name)}
			   .reduceByKey().countByKey();
```

Slide 67

```scala
 ….
 // Accumulator
 val countFR = sc.accumulator(0, "French")
 
 val people = sc.textFile("people.txt").map(_.split(","))
			.map(p => Person(p(0),p(1), p(2).trim.toInt))
 
 val country = sc.textFile("countries.txt").map(_.split(","))
			.map(c => Country(c(0),e(1))
 
 val countryBc = sc.broadcast(country)

 people.map(p => (p.countryCode, 1))
			  .join(countryBc.value.map(c => (c.code, c.name))
			  .filter { case (countryCode,(one,name)) => code == "FR"}
			  .reduceByKey().foreach{ case (code,iter) => countFR += iter.lenght};

 println("People in France : "+countFR);

```

Slide 72

```scala
 val sqlContext = new org.apache.spark.sql.SQLContext(sc);
 import sqlContext.createSchemaRDD;

```

Slide 72

```scala
 case class Person(name: String, age: Int)

 val people = sc.textFile("people.txt").map(_.split(",")).map(p => Person(p(0), p(1).trim.toInt))

 people.registerTempTable("people")
```

Slide 73

```scala
val teenagers: SchemaRDD = sqlContext.sql("SELECT name, age 
						   FROM people 
						   WHERE age ≥ 13 AND age ≤ 19");

 // or

 val teenagers: SchemaRDD = people.where('age ≥ 10).where('age ≤ 19).select('name);

 teenages.map(row => "Name : "+row(0)).collect().foreach(println)

```

Slide 76

```sql
 SELECT name, age 
 FROM people 
 WHERE age ≥ 13 AND age ≤ 19 

```

```sql
 SELECT name, age 
```

```sql
 WHERE age ≥ 13 AND age ≤ 19 
```

```scala
 val people:RDD[Person]
 val teenagers:RDD[(String,Int)]
    = people
    .filter(p => p.age ≥ 13 && p.age ≤ 19)
    .map(p => (p.name, p.age))
```

```scala
.map(p => (p.name, p.age))
```

```scala
.filter(p => p.age ≥ 13 && p.age ≤ 19)
```

Slide 77

```sql
 SELECT p.name, e.content,e.date 
 FROM people p JOIN emails e
 ON p.login = e.login
 WHERE p.age ≥ 28 
 AND p.age ≤ 32
 AND e.date ≥ ‘2015-01-01 00:00:00’

```

Slide 78

```scala
 val people:RDD[Person]
 val emails:RDD[Email]
 val p = people.map(p => (p.login, p)) 
 val e = emails.map(e => (e.login,e)
 val evedrops:RDD[(String,String,Date)]
    = p.join(e)
       .filter{ case(login,(p,e) => 
		      p.age ≥ 28 && p.age ≤ 32 &&
		      e.date ≥ ‘2015-01-01 00:00:00’ 
	     }
      .map{ case(login,(p,e)) => 
			    (p.name,e.content,e.date)
	    }
```

Slide 79

```scala
val p = people
	    .filter(p => p.age ≥ 28 && p.age ≤ 32 )
      .map(p => (p.login,p.name)) 
 val e = emails
	    .filter(e.date ≥ ‘2015-01-01 00:00:00’ )
	    .map(e => (e.login,(e.content,e.date))
 
 val evedrops:RDD[(String,String,Date)]
    = p.join(e)
      .map{case(login,(name,(content,date)) => 
			    (name,content,date)
	    }
```

Slide 84

```scala
 val ssc = new SparkStreamContext("local", "test")
 
 ssc.setBatchDuration(Seconds(1))

```

```scala
 val words = ssc.createNetworkStream("http://...")
 val ones = words.map(w => (w, 1))! 
 val freqs = ones.reduceByKey { case (count1, count2) => count1 + count2}
 freqs.print()

 // Start the stream computation
 ssc.run 
```

Slide 85

```scala
 val freqs = ones.reduceByKey { case (count1, count2) => count1 + count2}
 val freqs_60s = freqs.window(windowDuration = Seconds(60), 
 				slideDuration = Second(20))
			.reduceByKey { case (count1, count2) => count1 + count2}

 // or

 val freqs_60s = ones.reduceByKeyAndWindow(couple => couple._1+couple._2,
 						windowDuration = Seconds(60), 
 						slideDuration = Seconds(20)) 
```

Slide 87

```scala
 val freqs = ones.reduceByKey { case (count1, count2) => count1 + count2}
 val freqs_60s = freqs.window(windowDuration = Seconds(60), 
 				slideDuration = Second(20))
			.reduceByKey { case (count1, count2) => count1 + count2}

 // or

 val freqs_60s = ones.reduceByKeyAndWindow(couple => couple._1+couple._2, 
						couple => couple._1 - couple._2, //inverse
						windowDuration = Seconds(60), 
 						slideDuration = Seconds(20)) 
```

Slide 90

```scala
 val connection = createNewConnection()  //executed at the driver

 dstream.foreachRDD(rdd => {
      rdd.foreach(record => {
          connection.send(record) //executed at the worker
      })
  })

 connection.close() //executed at the driver
```

Slide 91

```scala
 dstream.foreachRDD(rdd => {
      rdd.foreach(record => {
         val connection = createNewConnection()  //executed at the worker
         connection.send(record) //executed at the worker
         connection.close() //executed at the worker
      })
  })
```

Slide 92

```scala
 dstream.foreachRDD(rdd => {
      rdd.foreach(record => {
         val connection = ConnectionPool.getConnection()
         connection.send(record)
         ConnectionPool.release(connection) 
      })
  })
```

Slide 93

```scala
 dstream.foreachRDD(rdd => {
      rdd.foreach(record => {
         ConnectionPool.withConnectionDo(connection => connection.send(record))
      })
  })
```

Slide xx

```scala

```

Slide 110

```scala
abstract class RDD[T](…) {

  	@DeveloperApi
	def compute(split: Partition, context: TaskContext): Iterator[T]


	protected def getPartitions: Array[Partition]


	protected def getPreferredLocations(split: Partition): Seq[String] = Nil			

 }
```

Slide 113

```scala
  connector.withSessionDo {
      session => session.execute("SELECT xxx FROM yyy").all()
  }
```

Slide 114

```scala
 // Import Cassandra-specific functions on SparkContext and RDD objects
 import com.datastax.driver.spark._


 // Spark connection options
 val conf = new SparkConf(true)
		    .setMaster("spark://192.168.123.10:7077")
		    .setAppName("cassandra-demo")
 		    .set("cassandra.connection.host", "192.168.123.10") // initial contact
        .set("cassandra.username", "cassandra")
		    .set("cassandra.password", "cassandra") 

 val sc = new SparkContext(conf)
```

Slide 115

```sql
	CREATE TABLE test.words (word text PRIMARY KEY, count int);

	INSERT INTO test.words (word, count) VALUES ('bar', 30);
	INSERT INTO test.words (word, count) VALUES ('foo', 20);

```

Slide 116

```scala
 // Use table as RDD
 val rdd = sc.cassandraTable("test", "words")
 // rdd: CassandraRDD[CassandraRow] = CassandraRDD[0]

 rdd.toArray.foreach(println)
 // CassandraRow[word: bar, count: 30]
 // CassandraRow[word: foo, count: 20]

 rdd.columnNames    // Stream(word, count) 
 rdd.size           // 2

 val firstRow = rdd.first  // firstRow: CassandraRow = CassandraRow[word: bar, count: 30]
 firstRow.getInt("count")  // Int = 30
```

Slide 117

```scala
val newRdd = sc.parallelize(Seq(("cat", 40), ("fox", 50)))
// newRdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[2]

newRdd.saveToCassandra("test", "words", Seq("word", "count"))
```

```sql
 SELECT * FROM test.words;

  word | count
 ------+-------
   bar |    30
   foo |    20
   cat |    40
   fox |    50
```

Slide xxx

```scala

```

Slide xxx

```scala

```

Slide xxx

```scala

```

Slide xxx

```scala

```
