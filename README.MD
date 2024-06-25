# MongoDB Task 02

1. This task is about designing a database for the Zen class program using MongoDB.

2. For this task we have created, 8 collections.
   1. Students
   2. codekata
   3. attendance
   4. topics
   5. tasks
   6. company_drives
   7. mentors
   8. Participation

    Each collection has 20 records in JSON format.

3. Find all the topics and tasks which are thought in the month of October.

   db.tasks.aggregate(
    [ 
{
$match: {
date: {
$gte:'2020-10-01T00:00:00Z',
$lte: '2020-10-31T23:59:59Z'
}}
},
{
$lookup: {
from: "topics",
localField: "topic_id",
foreignField: "user_id",
as: "topics"
}},
{
$unwind: "$topics"
},
{
$project: {
date: 1,
task_name: 1,
topic_name: "$topics.topic_name",
}}
])

    $match: $match in MongoDB lets you filter documents in an aggregation pipeline based on specific conditions, helping to focus on relevant data early in the processing.

    $lookup: $lookup in MongoDB is used to perform a left outer join between documents from two collections in the same database. It retrieves documents from another collection and adds them to the current documents as an array. This stage is particularly useful for combining data from multiple collections based on a common field.

    $unwind: $unwind in MongoDB is an aggregation pipeline stage that is used to deconstruct an array field from the input documents to output a document for each element. It essentially flattens an array field, creating multiple documents from a single document that contains arrays, one document for each element of the array. This stage is commonly used when you want to perform operations or further aggregations on elements within an array.

    $project: $project is an aggregation pipeline stage used to reshape documents. It allows you to include, exclude, rename, or create new fields in the output documents.

4. Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020.

    db.company_drives.find({
    drive_date: {
        $gte: "2020-10-15T00:00:00Z",
        $lte: "2020-10-31T23:59:59Z"
    }
})

    find(): find() method is used to query documents from a collection. It retrieves documents that match a specified query criteria.

5. Find all the company drives and students who are appeared for the placement.

    db.company_drives.aggregate([
    {
     $lookup: {
      from: "participation",
      localField: "drive_id",
      foreignField: "part_id",
      as: "placements"
    }
    },
   {
    $unwind: "$placements"
   },
   {
    $lookup: {
      from: "students",
      localField: "placements.student_id",
      foreignField: "drive_id",
      as: "student_info"
    }
   },
   {
    $unwind: "$student_info"
   },
   {
    $match: {
      "placements.status": "attended"
    }
   },
   {
    $project: {

      drive_name: "$drive_name",
      drive_date: "$drive_date",
      student_name: "$student_info.name",
      status:"$placements.status"
     }
     }
     ])

    Aggregate: Aggregate operation is used to process data records and return computed results. It is highly flexible and powerful, allowing for complex transformations and aggregations of data within a collection.

6. Find the number of problems solved by the user in codekata.

    db.students.aggregate([
    {
        $lookup: {
            from: "codekata",
            localField: "user_id",
            foreignField: "student_id",
            as: "codekata_info"
        }
    },
    {
        $unwind: "$codekata_info" 
    },
    {
        $project: {
            _id:0,
            name: 1,
            problems_solved: "$codekata_info.problems_solved",
            student_id: "$codekata_info.user_id",
        }}
        ])

7. Find all the mentors with who has the mentee's count more than 15.

     db.mentors.aggregate([
     {
        $match: {
            mentees_count: { $gt: 15 }
        }
    },
    {
        $project: {
            _id: 0,
            mentor_name_with_count: {
                $concat: [
                    "$mentor_name",
                    " - Mentees Count: ",
                    { $toString: "$mentees_count" }
                ]
            }
        }
      }
    ])

     $concat: In MongoDB's aggregation framework, $concat is an operator used to concatenate strings. It allows you to combine multiple strings or expressions into a single string.

     $toString: $toString is an expression operator that converts a value to its string representation. It's particularly useful when you need to ensure that a field or an expression within an aggregation pipeline is treated as a string.

8. Find the number of users who are absent and task is not submitted  between 15 oct-2020 and 31-oct-2020.

     db.attendance.aggregate (

      [
     {
     $match: {
      date_attended: {
        $gte: "2020-10-15T00:00:00Z",
        $lte: "2020-10-31T23:59:59Z"
      },
      status: "absent"
         }
        },
     {
     $lookup: {
      from: "tasks",
      let: { student_id: "$student_id" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$student_id", "$$student_id"] },
                { $gte: ["$date", "2020-10-15T00:00:00Z"] },
                { $lte: ["$date", "2020-10-31T23:59:59Z"] },
                { $ne: ["$task_status", "Completed"] }
               ]
             }
           }
         }
        ],
      as: "not_completed_tasks"
       }
      },
      {
       $match: {
      "not_completed_tasks": { $exists: true, $not: { $size: 0 } }
       }
       },
       {
       $count: "The number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020"
       }
       ])

9. [Task](./MongoDB%20Task%2002.pdf) has been submitted in pdf format. Click More pages to view the full PDF.



