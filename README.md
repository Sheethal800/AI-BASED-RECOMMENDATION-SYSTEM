# AI-BASED-RECOMMENDATION-SYSTEM-TASK 4

Name:SHEETHAL P V

Company:CODTECH IT SOLUTIONS 

ID:CT6WTDS627

DOMAIN:JAVA PROGRAMMING

DURATION-NOV 25th ,2024 to JAN 10th ,2025

MENTOR NAME-Neela Santhosh Kumar

Key Features and Objectives

User-Item Ratings:
We have a set of users who have rated movies. The ratings are stored in a Map<String, Map<String, Integer>> where the outer map represents users, and the inner map stores the ratings of individual movies.
Example: User1 has rated MovieA with a score of 5.

Cosine Similarity:
The cosine similarity is used to measure how similar two users are based on the movies they've both rated.

Recommendation Engine:
The recommendation engine suggests movies for a target user based on the ratings of similar users.
For a given target user (e.g., "User1"), we find other users with the highest cosine similarity, and recommend movies that the target user has not rated yet but are highly rated by these similar user

Recommendation system using apache mahout 

Prepare the data
user1,item1,4.0
user1,item2,5.0
user2,item1,3.0
user2,item3,4.0
user3,item2,2.0
user3,item3,5.0

Java code for the recommender import org.apache.mahout.cf.taste.impl.model.file.FileDataModel;
import org.apache.mahout.cf.taste.impl.recommender.GenericItemBasedRecommender;
import org.apache.mahout.cf.taste.impl.recommender.RecommendedItem;
import org.apache.mahout.cf.taste.impl.similarity.PearsonCorrelationSimilarity;
import org.apache.mahout.cf.taste.model.DataModel;
import org.apache.mahout.cf.taste.recommender.Recommender;
import org.apache.mahout.cf.taste.similarity.ItemSimilarity;

import java.io.File;
import java.io.IOException;
import java.util.List;

public class SimpleRecommender {

    public static void main(String[] args) {
        try {
            // Step 1: Load user-item ratings data
            File dataFile = new File("data.csv"); // Adjust the file path as needed
            DataModel model = new FileDataModel(dataFile);

            // Step 2: Calculate item similarities (using Pearson Correlation here)
            ItemSimilarity similarity = new PearsonCorrelationSimilarity(model);

            // Step 3: Create the recommender using the item similarity
            Recommender recommender = new GenericItemBasedRecommender(model, similarity);

            // Step 4: Generate recommendations for a user (user with ID 'user1')
            List<RecommendedItem> recommendations = recommender.recommend(1, 3); // Recommend 3 items

            // Step 5: Output the recommendations
            for (RecommendedItem recommendation : recommendations) {
                System.out.println("Recommended Item: " + recommendation.getItemID() +
                        " with predicted rating: " + recommendation.getValue());
            }

        } catch (IOException | org.apache.mahout.cf.taste.impl.common.TasteException e) {
            e.printStackTrace();
        }
    }
}

Recommendation engine and sample data 

import java.util.*;

class RecommendationEngine {
    // Sample dataset of users and their ratings for items
    private static Map<String, Map<String, Integer>> userItemRatings = new HashMap<>();

    // Function to add user ratings to the dataset
    public static void addRating(String user, String item, int rating) {
        userItemRatings.putIfAbsent(user, new HashMap<>());
        userItemRatings.get(user).put(item, rating);
    }

    // Function to compute cosine similarity between two items
    public static double cosineSimilarity(Map<String, Integer> item1Ratings, Map<String, Integer> item2Ratings) {
        Set<String> commonUsers = new HashSet<>(item1Ratings.keySet());
        commonUsers.retainAll(item2Ratings.keySet());

        if (commonUsers.isEmpty()) return 0;

        int dotProduct = 0;
        int item1Magnitude = 0;
        int item2Magnitude = 0;

        for (String user : commonUsers) {
            dotProduct += item1Ratings.get(user) * item2Ratings.get(user);
        }

        for (String user : item1Ratings.keySet()) {
            item1Magnitude += item1Ratings.get(user) * item1Ratings.get(user);
        }

        for (String user : item2Ratings.keySet()) {
            item2Magnitude += item2Ratings.get(user) * item2Ratings.get(user);
        }

        return dotProduct / (Math.sqrt(item1Magnitude) * Math.sqrt(item2Magnitude));
    }

    // Function to get recommendations for a user
    public static List<String> getRecommendations(String user) {
        Map<String, Integer> userRatings = userItemRatings.get(user);
        if (userRatings == null || userRatings.isEmpty()) return Collections.emptyList();

        Map<String, Double> itemSimilarities = new HashMap<>();
        
        // Compare each item the user has rated with all other items
        for (String ratedItem : userRatings.keySet()) {
            for (String otherItem : userItemRatings.keySet()) {
                if (ratedItem.equals(otherItem)) continue;

                Map<String, Integer> ratedItemRatings = new HashMap<>();
                Map<String, Integer> otherItemRatings = new HashMap<>();
                
                // Create maps for the ratings of each item
                for (String u : userItemRatings.keySet()) {
                    if (userItemRatings.get(u).containsKey(ratedItem)) {
                        ratedItemRatings.put(u, userItemRatings.get(u).get(ratedItem));
                    }
                    if (userItemRatings.get(u).containsKey(otherItem)) {
                        otherItemRatings.put(u, userItemRatings.get(u).get(otherItem));
                    }
                }

                double similarity = cosineSimilarity(ratedItemRatings, otherItemRatings);
                itemSimilarities.put(otherItem, similarity);
            }
        }

        // Sort items by similarity (descending order)
        List<Map.Entry<String, Double>> itemList = new ArrayList<>(itemSimilarities.entrySet());
        itemList.sort((e1, e2) -> Double.compare(e2.getValue(), e1.getValue()));

        // Return the top 5 recommended items
        List<String> recommendations = new ArrayList<>();
        for (int i = 0; i < Math.min(5, itemList.size()); i++) {
            recommendations.add(itemList.get(i).getKey());
        }

        return recommendations;
    }

    // Sample data initialization
    public static void initSampleData() {
        addRating("Alice", "Item1", 5);
        addRating("Alice", "Item2", 3);
        addRating("Alice", "Item3", 4);
        addRating("Bob", "Item1", 4);
        addRating("Bob", "Item2", 5);
        addRating("Bob", "Item3", 2);
        addRating("Carol", "Item1", 2);
        addRating("Carol", "Item2", 4);
        addRating("Carol", "Item3", 5);
    }

    public static void main(String[] args) {
        // Initialize the sample data
        initSampleData();

        // Test the recommendation engine for Alice
        String user = "Alice";
        List<String> recommendations = getRecommendations(user);

        System.out.println("Recommended items for " + user + ": " + recommendations);
    }
}
