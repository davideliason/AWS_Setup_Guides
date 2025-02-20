Step    Details
1   An Amazon CloudWatch Events event calls the salesAnalysisReport Lambda function at 8 PM every day Monday through Saturday.
2   The salesAnalysisReport Lambda function invokes another Lambda function, salesAnalysisReportDataExtractor, to retrieve the report data.
3   The salesAnalysisReportDataExtractor function runs an analytical query against the café database (cafe_db).
4   The query result is returned to the salesAnalysisReport function.
5   The salesAnalysisReport function formats the report into a message and publishes it to the salesAnalysisReportTopic Amazon Simple Notification Service (Amazon SNS) topic.
6   The salesAnalysisReportTopic SNS topic sends the message by email to the administrator.


