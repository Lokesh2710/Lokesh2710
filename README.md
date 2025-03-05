Namaste 🙏 (Hello 👋)!

I’m Lokesh Kirad, from Neemuch District, Madhya Pradesh, India. I am passionate about Cloud Computing and Full-Stack Web Development.

 Educational Background :
- Master of Computer Applications (M.C.A.) from Hyderabad Central University (July 2018 - June 2021)
  - [Hyderabad Central University Website](http://acad.uohyd.ac.in/)

 Languages Known : Hindi , English

 Technical Skills :
 Programming Languages : HTML ,JavaScript ,CSS (Basics) ,TypeScript ,Java
 Web Development Frameworks :- Angular.js , Spring Boot
 Databases : MySQL, PostgreSQL
 Version Control : GitHub ,Bitbucket
 DevOps Tools : Jenkins , Bamboo , Docker
 Cloud Computing : AWS (Basics)
 Project Management/Agile Tools: Jira

 Connect with Me:
- [LinkedIn Profile](https://www.linkedin.com/in/lokeshkk/)

I look forward to exploring opportunities in cloud computing and full-stack web development. Let’s connect and collaborate!


import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedClient;
import software.amazon.awssdk.enhanced.dynamodb.DynamoDbTable;
import software.amazon.awssdk.enhanced.dynamodb.TableSchema;
import software.amazon.awssdk.enhanced.dynamodb.model.BatchGetItemEnhancedRequest;
import software.amazon.awssdk.enhanced.dynamodb.model.ReadBatch;

@ExtendWith(MockitoExtension.class)
class W4RWithholdingPercentagesRepositoryTest {

    @Mock
    private DynamoDbConfiguration dynamoDbConfiguration;

    @Mock
    private DynamoDbEnhancedClient dynamoDbEnhancedClient;

    @Mock
    private DynamoDbTable<W4RWithholdingPercentages> table;

    @InjectMocks
    private W4RWithholdingPercentagesRepository repository;

    @BeforeEach
    void setUp() {
        when(dynamoDbConfiguration.dynamoDBEnhancedClient()).thenReturn(dynamoDbEnhancedClient);
        when(dynamoDbEnhancedClient.table("W4RWithholdingPercentages", TableSchema.fromBean(W4RWithholdingPercentages.class)))
                .thenReturn(table);
    }

    @Test
    void testGetW4RPercentages_WithNullAccountIds_ReturnsEmptyList() {
        List<W4RWithholdingPercentages> result = repository.getW4RPercentages(null, "SALE");
        assertNotNull(result);
        assertTrue(result.isEmpty());
    }

    @Test
    void testGetW4RPercentages_WithEmptyAccountIds_ReturnsEmptyList() {
        List<W4RWithholdingPercentages> result = repository.getW4RPercentages(new ArrayList<>(), "SALE");
        assertNotNull(result);
        assertTrue(result.isEmpty());
    }

    @Test
    void testGetW4RPercentages_WithValidAccountIds_ReturnsData() {
        List<String> accountIds = List.of("123", "456");
        String transactionType = "SALE";

        List<W4RWithholdingPercentages> mockResults = List.of(new W4RWithholdingPercentages(), new W4RWithholdingPercentages());

        // Simulating batch get results
        when(table.getItem(any())).thenReturn(mockResults.get(0), mockResults.get(1));

        // Mock batchGetItem to return the expected items
        when(dynamoDbEnhancedClient.batchGetItem(any(BatchGetItemEnhancedRequest.class)))
                .thenReturn(Map.of(table, Set.copyOf(mockResults))); // Avoiding BatchGetResult

        List<W4RWithholdingPercentages> result = repository.getW4RPercentages(accountIds, transactionType);

        assertNotNull(result);
        assertEquals(2, result.size());
    }

    @Test
    void testGetW4RPercentages_WithReadBatchCreation() {
        List<String> accountIds = List.of("789");
        String transactionType = "PURCHASE";

        when(dynamoDbEnhancedClient.batchGetItem(any(BatchGetItemEnhancedRequest.class)))
                .thenReturn(Map.of(table, Set.of(new W4RWithholdingPercentages())));

        List<W4RWithholdingPercentages> result = repository.getW4RPercentages(accountIds, transactionType);

        assertNotNull(result);
        assertEquals(1, result.size());

        verify(dynamoDbEnhancedClient, times(1)).batchGetItem(any(BatchGetItemEnhancedRequest.class));
    }

    @Test
    void testGetW4RPercentages_ExceptionThrown() {
        when(dynamoDbConfiguration.dynamoDBEnhancedClient()).thenThrow(new RuntimeException("DynamoDB error"));

        RuntimeException exception = assertThrows(RuntimeException.class, () -> {
            repository.getW4RPercentages(List.of("123"), "SALE");
        });

        assertTrue(exception.getMessage().contains("Getting error while fetching data"));
    }
}

