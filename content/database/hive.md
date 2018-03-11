---
title: "hive"
date: 2018-01-21 17:15
---

# Hive 排序
## sort by
局部排序，每一个 reducer 结果之内是有序的, 但是多个 reducer 之间是无序的；
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee sort by salary;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```
## order by
全局排序，每一个 reducer 结果之内是有序的额, reducer 与 reducer 之间是有序的，结果可以直接拼接在一起构成全序；
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee order by salary; 
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```
## distribute by
组间排序，每一个 reducer 结果之内无序， reducer 与 reducer 之间有序；
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee distribute by id;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 3            | mahesh         | 25            | 25000            | HR                   |
+--------------+----------------+---------------+------------------+----------------------+--+
```

## distribute by & sort by
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee distribute by id sort by salary;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+

```
## cluster by
只针对某个列的 distribute by col + sort by col;
```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee distribute by id sort by id;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```

```sql
#set the number of reducers to 2
0: jdbc:hive2://localhost:10000> set mapred.reduce.tasks=2;

0: jdbc:hive2://localhost:10000> select * from employee cluster by id;
+--------------+----------------+---------------+------------------+----------------------+--+
| employee.id  | employee.name  | employee.age  | employee.salary  | employee.department  |
+--------------+----------------+---------------+------------------+----------------------+--+
| 2            | sakshi         | 22            | 60000            | HR                   |
| 10002        | rahul          | 23            | 250000           | BIGDATA              |
| 1            | aarti          | 28            | 55000            | HR                   |
| 3            | mahesh         | 25            | 25000            | HR                   |
| 10001        | rajesh         | 29            | 50000            | BIGDATA              |
| 10003        | dinesh         | 35            | 70000            | BIGDATA              |
+--------------+----------------+---------------+------------------+----------------------+--+
```

## 上述四种排序对应的 map reduce 实现方式
### sort by
mapreduce 实现局部排序
```python
--------------------Map Phase---------------------------------|------------------------------Reduce Phase----------------------
(map    ->    partition    ->    sort    ->  |-----------|)   ->    (shuffle    ->    merge sort    ->    group    ->    reduce)
 |              |                |           |split merge|                              |                  |               |
 |              |                |           |sort       |                              |                  |               |
map()       Partitioner     SortComparator                                         SortComparator    GroupComparator    reduce()

```
其实 MapReduce 本身就已经是默认局部排序了， 只要实现自己的 key 的 Partitioner SortComparator 和 GroupComparator 即可自动 reduce 内部排序

### order by
mapreduce 实现全序排序

第一种方法是在正常的排序完成之后, 对第一阶段每一个 reducer 结果再次构造 MapReduce，但是该阶段只设置 一个 reducer， 那最后合并到一个reducer文件即可，但是当数量量很大，这个会导致某个reducer 压力很大；

第二种方法就是，在操作partition 的时候， 让 reducer 组间 有序， 即 根据需要改写 Partitioner， 根据 key 范围进行划分不同的 reducer，保证 reducer 之间有序； 这个可以自己实现，但是hadoop 已经提供了一个 TotalOrderPartitioner 了， 采用 trie 树 采样达到负载均衡；

### distribute by
MapReduce 实现组间排序
只需要实现自定义的 Partitioner 即可实现组间排序；

### cluster by
MapReduce 实现 聚集排序
同时实现自定义 Partitioner SortComparator GroupComparator 即可；

### secondary sort
MapReduce 多级排序实现；

自定义复合Key值 CompositeKey， 按需要实现 自定义 Partitioner SortComparator GroupComparator

# MapReduce Join data 实现
## Reduce-Side Join [Repartition join case study](http://blog.ditullio.fr/2016/01/29/hadoop-basics-repartition-join-mapreduce/)
### 连接操作类型 mysql join type
四个基本的join，其实还有三种。详细可以参考 [mysql join]()
- inner join
`SELECT <field_list> FROM TABLE_A A INNER JOIN TABLE_B B ON A.KEY = B.KEY`
- left join
`SELECT <field_list> FROM TABLE_A A LEFT JOIN TABLE_B B ON A.KEY = B.KEY`
- right join
`SELECT <field_list> FROM TABLE_A A RIGHT JOIN TABLE_B B ON A.KEY = B.KEY`
- outer join
`SELECT <field_list> FROM TABLE_A A FULL OUTER JOIN TABLE_B B ON A.KEY = B.KEY`

### 数据集 dataset
两张表，donation 和 project。描述的是项目捐赠情况。donation 主要是捐款人信息，project 主要是受助人信息。
- Donation attributes : id, project id, donor city, date, total
- Project attributes : id, school city, poverty level, primary subject

### 操作步骤
#### Map 阶段
将需要连接的表通过map之后，输出 连接主键和其他信息 的(outputKey, outputValue)。即 (joinKey, joinInfo)。这里的 joinKey 可以是 Text 类型也可以是 Writable 类型(可以序列化的对象)，outputValue 也可以是 纯文本Text 形式也可以是 Writable 对象，不同的方式会影响数据读写效率。Text 的效率将会更高，因为不需要 serialization/deserialization.

#### Reduce 阶段
不同表中相同的joinKey会被 partition 到相同的 Reduce 分区。在 reduce 里面，就可以将两个表的 joinInfo 连接起来，这个时候根据不用的 join 类型进行不同的连接操作。

具体操作就是，使用两个列表在内存中，存储两个不同表的 joinInfo。 由于不同表的相同 joinKey joinInfo 被聚集到同一个 reduce 的 values 列表中，如何区分出不同表的 joinInfo 进而继续连接呢，这和 joinInfo 使用什么类型有关系，使用 Text，你就需要在 Map 阶段加入一些标记符号来区分，如果使用自定义的 Writable 对象，那就可以直接 instance of 来判断了。

```java
		protected void reduce(Text projectId, Iterable<ObjectWritable> values, Context context) throws IOException, InterruptedException {

			// Clear data lists
			donationsList.clear();
			projectsList.clear();

			// Fill up data lists with selected fields
			for (ObjectWritable value : values) {
				Object object = value.get();

				if (object instanceof DonationWritable) {
					DonationWritable donation = (DonationWritable) object;
					String donationOutput = String.format("%s|%s|%s|%s|%.2f", donation.donation_id, donation.project_id, 
							donation.donor_city, donation.ddate, donation.total);
					donationsList.add(donationOutput);

				} else if (object instanceof ProjectWritable) {
					ProjectWritable project = (ProjectWritable) object;
					String projectOutput = String.format("%s|%s|%s|%s", project.project_id, project.school_city, 
							project.poverty_level, project.primary_focus_subject);
					projectsList.add(projectOutput);

				} else {
					String errorMsg = String.format("Object of class %s is neither a %s nor %s.", 
							object.getClass().getName(), ProjectWritable.class.getName(), DonationWritable.class.getName());
					throw new IOException(errorMsg);
				}
			}


			// Join data lists (example with FULL OUTER JOIN)
			if (!donationsList.isEmpty()) {

				for (String dontationStr : donationsList) {

					if (!projectsList.isEmpty()) {

						// Case 1 : Both LEFT and RIGHT sides of the join have values
						// Extra loop to write all combinations of (LEFT, RIGHT)
						// These are also the outputs of an INNER JOIN
						for (String projectStr : projectsList) {
							donationOutput.set(dontationStr);
							projectOutput.set(projectStr);
							context.write(donationOutput, projectOutput);
						}

					} else {

						// Case 2 : LEFT side has values but RIGHT side doesn't.
						// Simply write (LEFT, null) to output for each value of LEFT.
						// These are also the outputs of a LEFT OUTER JOIN
						donationOutput.set(dontationStr);
						projectOutput.set(NULL_PROJECT_OUTPUT);
						context.write(donationOutput, projectOutput);
					}
				}

			} else {

				// Case 3 : LEFT side doesn't have values, but RIGHT side has values
				// Simply write (null, RIGHT) to output for each value of LEFT.
				// These are also the outputs of a RIGHT OUTER JOIN
				for (String projectStr : projectsList) {
					donationOutput.set(NULL_DONATION_OUTPUT);
					projectOutput.set(projectStr);
					context.write(donationOutput, projectOutput);
				}
			}
		}
```
### 优化1
使用 projection 在map 阶段将不需要的字段过滤掉。
```java
public static class DonationsMapper extends Mapper<Object, DonationWritable, Text, ObjectWritable> {

		private Text outputKey = new Text();
		private ObjectWritable outputValue = new ObjectWritable();

		@Override
		public void map(Object key, DonationWritable donation, Context context) throws IOException, InterruptedException {

			outputKey.set(donation.project_id);
			// Create new object with projected values
			outputValue.set(DonationProjection.makeProjection(donation));
			context.write(outputKey, outputValue);
		}
	}

	/**
	 * Projects Mapper.
	 *
	 */
	public static class ProjectsMapper extends Mapper<Object, ProjectWritable, Text, ObjectWritable> {

		private Text outputKey = new Text();
		private ObjectWritable outputValue = new ObjectWritable();

		@Override
		public void map(Object offset, ProjectWritable project, Context context) throws IOException, InterruptedException {

			outputKey.set(project.project_id);
			// Create new object with projected values
			outputValue.set(ProjectProjection.makeProjection(project));
			context.write(outputKey, outputValue);

		}
	}

```
### 优化2
使用 纯Text 形式。

```java
public static class JoinReducer extends Reducer<Text, Text, Text, Text> {

		private Text donationOutput = new Text();
		private Text projectOutput = new Text();

		private List<String> donationsList;
		private List<String> projectsList;

		@Override
		protected void reduce(Text projectId, Iterable<Text> values, Context context) throws IOException, InterruptedException {

			// Clear data lists
			donationsList = new ArrayList<>();
			projectsList = new ArrayList<>();

			// Fill up data lists with selected fields
			for (Text value : values) {
				String textVal = value.toString();

				// Get first char which determines the type of data
				char type = textVal.charAt(0);

				// Remove the type flag "P|" or "D|" from the beginning to get original data content
				textVal = textVal.substring(2);

				if (type == 'D') {
					donationsList.add(textVal);

				} else if (type == 'P') {
					projectsList.add(textVal);

				} else {
					String errorMsg = String.format("Type is neither a D nor P.");
					throw new IOException(errorMsg);
				}
			}

			// Join data lists only if both sides exist (INNER JOIN)
			if (!donationsList.isEmpty() && !projectsList.isEmpty()) {

				// Write all combinations of (LEFT, RIGHT) values
				for (String dontationStr : donationsList) {
					for (String projectStr : projectsList) {
						donationOutput.set(dontationStr);
						projectOutput.set(projectStr);
						context.write(donationOutput, projectOutput);
					}
				}
			}

		}		
	}	
```

## Map-Side Join [Replicated join case study](http://blog.ditullio.fr/2016/02/03/hadoop-basics-replicated-join-in-mapreduce/)
### 局限性
1. 小表能够刷入Map端JVM 内存之中
2. 只能以大表为准做　inner join 或者 left join. 因为小表会被复制到每一个map,因此每一个map 都能看到全局的做连接的右边的小表，但是反过来小表无法看到全局的大表;

### 基本操作
小表刷入每一个Mapper 的内存，装入　HashMap 之中，之后就可以在 map 函数之中使用其作为右表做inner join 或者　left join 了；这个版本基于　Writable 对象来做，通过从已经做好的SequenceFile中读取　Writable 类型的value

```java
public static class ReplicatedJoinMapper extends Mapper<Object, DonationWritable, Text, Text> {

		public static final String PROJECTS_FILENAME_CONF_KEY = "projects.filename";

		private Map<String, ProjectWritable> projectsCache = new HashMap<>();

		private Text outputKey = new Text();
		private Text outputValue = new Text();

		@Override
		public void setup(Context context) throws IOException, InterruptedException {

			boolean cacheOK = false;

			URI[] cacheFiles = context.getCacheFiles();
			final String distributedCacheFilename = context.getConfiguration().get(PROJECTS_FILENAME_CONF_KEY);

			Configuration conf = new Configuration();
			for (URI cacheFile : cacheFiles) {
				Path path = new Path(cacheFile);
				if (path.getName().equals(distributedCacheFilename)) {
					LOG.info("Starting to build cache from : " + cacheFile);
					try (SequenceFile.Reader reader = new SequenceFile.Reader(conf, SequenceFile.Reader.file(path))) 
					{
						LOG.info("Compressed ? " + reader.isBlockCompressed());
						Text tempKey = new Text();
						ProjectWritable tempValue = new ProjectWritable();

						while (reader.next(tempKey, tempValue)) {
							// Clone the writable otherwise all map values will be the same reference to tempValue
							ProjectWritable project = WritableUtils.clone(tempValue, conf);
							projectsCache.put(tempKey.toString(), project);
						}
					}
					LOG.info("Finished to build cache. Number of entries : " + projectsCache.size());

					cacheOK = true;
					break;
				}
			}

			if (!cacheOK) {
				LOG.error("Distributed cache file not found : " + distributedCacheFilename);
				throw new IOException("Distributed cache file not found : " + distributedCacheFilename);
			}
		}

		@Override
		public void map(Object key, DonationWritable donation, Context context)
				throws IOException, InterruptedException {

			ProjectWritable project = projectsCache.get(donation.project_id);

			// Ignore if the corresponding entry doesn't exist in the projects data (INNER JOIN)
			if (project == null) {
				return;
			}

			String donationOutput = String.format("%s|%s|%s|%s|%.2f", donation.donation_id, donation.project_id, 
					donation.donor_city, donation.ddate, donation.total);

			String projectOutput = String.format("%s|%s|%d|%s", 
					project.project_id, project.school_city, project.poverty_level, project.primary_focus_subject);

			outputKey.set(donationOutput);
			outputValue.set(projectOutput);
			context.write(outputKey, outputValue);
		}
	}
```

### 优化1
使用 projection 在map 阶段将不需要的字段过滤掉。
```java
/**
	 * Smaller "Project" class used for projection.
	 * 
	 * @author Nicomak
	 *
	 */
	public static class ProjectProjection {

		public String project_id;
		public String school_city;
		public String poverty_level;
		public String primary_focus_subject;

		public ProjectProjection(ProjectWritable project) {
			this.project_id = project.project_id;
			this.school_city = project.school_city;
			this.poverty_level = project.poverty_level;
			this.primary_focus_subject = project.primary_focus_subject;
		}
	}

	public static class ReplicatedJoinMapper extends Mapper<Object, DonationWritable, Text, Text> {

		private Map<String, ProjectProjection> projectsCache = new HashMap<>();

		private Text outputKey = new Text();
		private Text outputValue = new Text();

		@Override
		public void setup(Context context) throws IOException, InterruptedException {

			boolean cacheOK = false;

			URI[] cacheFiles = context.getCacheFiles();
			final String distributedCacheFilename = context.getConfiguration().get(PROJECTS_FILENAME_CONF_KEY);

			Configuration conf = new Configuration();
			for (URI cacheFile : cacheFiles) {
				Path path = new Path(cacheFile);
				if (path.getName().equals(distributedCacheFilename)) {
					LOG.info("Starting to build cache from : " + cacheFile);
					try (SequenceFile.Reader reader = new SequenceFile.Reader(conf, SequenceFile.Reader.file(path))) 
					{
						LOG.info("Compressed ? " + reader.isBlockCompressed());
						Text tempKey = new Text();
						ProjectWritable tempValue = new ProjectWritable();

						while (reader.next(tempKey, tempValue)) {

							// We are creating a smaller projection object here to save memory space.
							ProjectProjection projection = new ProjectProjection(tempValue);
							projectsCache.put(tempKey.toString(), projection);
						}
					}
					LOG.info("Finished to build cache. Number of entries : " + projectsCache.size());

					cacheOK = true;
					break;
				}
			}

			if (!cacheOK) {
				LOG.error("Distributed cache file not found : " + distributedCacheFilename);
				throw new IOException("Distributed cache file not found : " + distributedCacheFilename);
			}
		}

		@Override
		public void map(Object key, DonationWritable donation, Context context)
				throws IOException, InterruptedException {

			ProjectProjection project = projectsCache.get(donation.project_id);

			// Ignore if the corresponding entry doesn't exist in the projects data (INNER JOIN)
			if (project == null) {
				return;
			}

			if (project != null) {
				String donationOutput = String.format("%s|%s|%s|%s|%.2f", donation.donation_id, donation.project_id, 
						donation.donor_city, donation.ddate, donation.total);

				String projectOutput = String.format("%s|%s|%s|%s", 
						project.project_id, project.school_city, project.poverty_level, project.primary_focus_subject);

				outputKey.set(donationOutput);
				outputValue.set(projectOutput);
				context.write(outputKey, outputValue);
			}
		}
	}
```

### 优化２
纯文本形式

```java
public static class ReplicatedJoinMapper extends Mapper<Object, DonationWritable, Text, Text> {

		public static final String PROJECTS_FILENAME_CONF_KEY = "projects.filename";

		private Map<String, String> projectsCache = new HashMap<>();

		private Text outputKey = new Text();
		private Text outputValue = new Text();

		@Override
		public void setup(Context context) throws IOException, InterruptedException {

			boolean cacheOK = false;

			URI[] cacheFiles = context.getCacheFiles();
			final String distributedCacheFilename = context.getConfiguration().get(PROJECTS_FILENAME_CONF_KEY);

			Configuration conf = new Configuration();
			for (URI cacheFile : cacheFiles) {
				Path path = new Path(cacheFile);
				if (path.getName().equals(distributedCacheFilename)) {
					LOG.info("Starting to build cache from : " + cacheFile);
					try (SequenceFile.Reader reader = new SequenceFile.Reader(conf, SequenceFile.Reader.file(path))) 
					{
						LOG.info("Compressed ? " + reader.isBlockCompressed());
						Text tempKey = new Text();
						ProjectWritable tempValue = new ProjectWritable();

						while (reader.next(tempKey, tempValue)) {
							// Serialize important value to a string containing pipe-separated values
							String projectString = String.format("%s|%s|%s|%s", tempValue.project_id, 
									tempValue.school_city, tempValue.poverty_level, tempValue.primary_focus_subject);
							projectsCache.put(tempKey.toString(), projectString);
						}
					}
					LOG.info("Finished to build cache. Number of entries : " + projectsCache.size());

					cacheOK = true;
					break;
				}
			}

			if (!cacheOK) {
				LOG.error("Distributed cache file not found : " + distributedCacheFilename);
				throw new IOException("Distributed cache file not found : " + distributedCacheFilename);
			}
		}

		@Override
		public void map(Object key, DonationWritable donation, Context context)
				throws IOException, InterruptedException {

			String projectOutput = projectsCache.get(donation.project_id);

			// Ignore if the corresponding entry doesn't exist in the projects data (INNER JOIN)
			if (projectOutput == null) {
				return;
			}

			String donationOutput = String.format("%s|%s|%s|%s|%.2f", donation.donation_id, donation.project_id, 
					donation.donor_city, donation.ddate, donation.total);

			outputKey.set(donationOutput);
			outputValue.set(projectOutput);
			context.write(outputKey, outputValue);
		}
	}
```