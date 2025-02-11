
WordCount-Using-MapReduce-Hadoop

This repository contains a Hadoop MapReduce implementation for counting word frequencies in a text file. The project demonstrates how to process large datasets using Hadoop's distributed computing framework and provides a hands-on experience with Hadoop's ecosystem, including HDFS, MapReduce, and Docker.

-------------------

Project Overview

The goal of this project is to implement a Word Count program using Hadoop MapReduce. The program processes a text file, counts the occurrences of each word, and outputs the results in descending order of frequency. This project is designed to help students understand Hadoop's architecture, build and deploy MapReduce jobs, and interact with the Hadoop ecosystem using Docker.

------------
 Approach and Implementation

 Mapper Logic
The Mapper processes each line of the input text file. It performs the following steps:
1. Splits the input line into individual words.
2. Cleans the words by removing non-alphabetic characters and converting them to lowercase.
3. Emits a key-value pair for each word, where the key is the cleaned word and the value is 1 (indicating one occurrence of the word).

public class WordMapper extends Mapper<Object, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    @Override
    protected void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        StringTokenizer tokenizer = new StringTokenizer(value.toString());
        while (tokenizer.hasMoreTokens()) {
            word.set(tokenizer.nextToken().replaceAll("[^a-zA-Z]", "").toLowerCase());
            if (!word.toString().isEmpty()) {
                context.write(word, one);
            }
        }
    }
}

-----------------
 Reducer Logic
The Reducer aggregates the counts for each word and sorts the output in descending order of frequency. It performs the following steps:
1. Receives key-value pairs where the key is a word and the value is a list of counts (all 1s).
2. Sums up the counts for each word and stores them in a HashMap.
3. In the cleanup method, sorts the words by their frequency in descending order.
4. Emits the sorted words and their counts.

public class WordReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    private Map<Text, Integer> wordCountMap = new HashMap<>();

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        wordCountMap.put(new Text(key), sum);
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
        // Sort the words by frequency in descending order
        List<Map.Entry<Text, Integer>> sortedList = new ArrayList<>(wordCountMap.entrySet());
        sortedList.sort((a, b) -> b.getValue().compareTo(a.getValue()));

        // Emit the sorted results
        for (Map.Entry<Text, Integer> entry : sortedList) {
            context.write(entry.getKey(), new IntWritable(entry.getValue()));
        }
    }
}


-------------------
 Execution Steps

 1. Start the Hadoop Cluster
Run the following command to start the Hadoop cluster using Docker:
bash
docker compose up -d


 2. Build the Code
Build the project using Maven:
bash
mvn install


 3. Move JAR File to Shared Folder
Move the generated JAR file to the shared folder:
bash
mv target/.jar shared-folder/input/code/


 4. Copy JAR to Docker Container
Copy the JAR file to the Hadoop ResourceManager container:
bash
docker cp shared-folder/input/code/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/


 5. Move Dataset to Docker Container
Copy the input dataset to the Hadoop ResourceManager container:
bash
docker cp shared-folder/input/data/input.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/


 6. Connect to Docker Container
Access the Hadoop ResourceManager container:
bash
docker exec -it resourcemanager /bin/bash


Navigate to the Hadoop directory:
bash
cd /opt/hadoop-3.2.1/share/hadoop/mapreduce/


 7. Set Up HDFS
Create a folder in HDFS for the input dataset:
bash
hadoop fs -mkdir -p /input/dataset


Copy the input dataset to the HDFS folder:
bash
hadoop fs -put ./input.txt /input/dataset


 8. Execute the MapReduce Job
Run the MapReduce job using the following command:
bash
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/WordCountUsingHadoop-0.0.1-SNAPSHOT.jar com.example.controller.Controller /input/dataset/input.txt /output


 9. View the Output
To view the output of the MapReduce job:
bash
hadoop fs -cat /output/


 10. Copy Output from HDFS to Local OS
Copy the output from HDFS to your local machine:
1. Copy from HDFS:
    bash
    hdfs dfs -get /output /opt/hadoop-3.2.1/share/hadoop/mapreduce/
    
2. Copy from the container to your local machine:
    bash
    exit
    docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output/ shared-folder/output/
    

----------------

 Challenges Faced & Solutions

 Challenge 1: Docker Container Connectivity
- Issue: The Docker container was not accessible after starting the Hadoop cluster.
- Solution: Ensured that the Docker network was properly configured and used the correct container name to access it.

 Challenge 2: HDFS Permission Denied
- Issue: Received a "Permission Denied" error while copying files to HDFS.
- Solution: Used the hadoop fs -chmod command to change the permissions of the HDFS directory.

 Challenge 3: Sorting in Reducer
- Issue: Sorting the output by frequency in descending order was initially challenging.
- Solution: Implemented a HashMap to store word counts and used a custom sorting logic in the cleanup method of the Reducer.



--------------------
 Sample Input and Output

 Input

Hello world
Hello Hadoop
Hadoop is powerful
Hadoop is used for big data


Expected Output

hadoop 3
hello 2
is 2
used 1
for 1
big 1
data 1
powerful 1
world 1


------------------
Conclusion
This project provides a hands-on experience with Hadoop MapReduce, Docker, and HDFS. By completing this activity, students gain a deeper understanding of distributed computing and big data processing. The repository is self-explanatory and well-documented, making it easy for others to replicate and understand the implementation. The addition of sorting in the Reducer ensures the output is organized by word frequency, enhancing the usability of the results.
