This course is designed to help you build a solid introduction to Spring Batch and show you how to use it with Spring Boot. Our Spring experts guide you through building and running a fully functional batch application that generates billing reports for an imaginary cell phone company called Spring Cellular.

## What Will You Learn?

By the end of this course, you'll learn to:

@@@checks
{
"list": "Understand the **basics of batch processing with Spring Batch** || **Identify use-cases** where batch processing would be helpful || **Create**, **run**, and **test** batch jobs || Structure batch jobs into **workflows of steps** || Create **fault-tolerant** batch jobs"
}
@@@

## What Will You Build?

In the Labs of this course, you'll create a batch application that generates billing reports for an imaginary cell phone company called Spring Cellular. The application stores billing information in a relational database and generates a billing report. This application is based on Spring Boot and uses Spring Batch's features to create a robust batch processing system that is restartable and fault-tolerant.

You'll implement a batch job named `BillingJob` that is designed as follows:

![Billing Job](https://raw.githubusercontent.com/spring-academy/spring-academy-assets/main/courses/course-spring-batch-essentials/intro-lesson-billing-job.svg)

The billing job is structured in the following steps:

- **File preparation step**: copies the file that contains the monthly usage for Spring Cellular's customers from a file server to a staging area.
- **File ingestion step**: ingests the file into a relational database table that contains the data used to generate the billing report.
- **Report generation step**: processes the billing information from the database table, and generates a flat file that contains data for the customers who have spent more than $150.00 USD.

## Hands-On Labs

The Labs in this course provide an interactive terminal and editor, so you don't need any specific tools installed on your own machine.

## Prerequisites

In order to get the most out of this course, you should have:

@@@checks
{
"list": "Experience with Java || Familiarity with Spring Framework and Spring Boot || Basic knowledge of Docker and SQL"
}
@@@

## Project Based

Rather than structuring this course like a formal documentation or a textbook, we've approached it as a real-world development project. So what does this mean?

Each lesson is designed to explain a **specific** Spring Batch concept, and has a companion Lab representing a task in the development process that you might encounter during a real world project.

Additionally, rather than covering each concept in depth, we'll only cover enough details to complete each task, as well as help you generally understand what's going on "under the hood" in the application. We'll also explain **why** we're making the choices we make, and what **other options** and **trade-offs** exist.
