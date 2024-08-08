---
layout: post
title: "Investigating Concurrency Control in Ruby on Rails, with PostgreSQL"
date: 2024-05-27 10:04:00 +0200
categories: database
---

Recently I read a lot of database related resources. One topic that I saw often was data integrity, so I decided to take a closer look. 

A moderately busy web application can process hundreds of HTTP requests at once. These concurrent requests can make changes to the same data in the database, which can cause anomalies. Some commonly known anomalies are dirty read, unrepeatable read, phantom records, lost update and  write-skew, but other kinds of anomalies are also possible. If these anomalies occur, they cause data corruption, which is hard to detect, but seriously endangers the integrity of our data.

To better understand these anomalies, I create a demo application to showcase a classical write-skew anomaly. This Ruby on Rails application manages doctors on duty. It has a doctors table, with a `name` string and an `on_duty` boolean value. This boolean represents if the given doctor is currently on duty or not. The doctors can use the UI to check in and out from duty. 

![doctors-on-duty](/images/concurrency-control/doctors-on-duty-2.png)

This system has one really important business rule, **at least one doctor must be on duty all the time**. To enforce this, the I added a check to raise an error and prevent the doctor from checking out, if not enough doctors would stay on duty:

```ruby
# app/models/doctor.rb

class Doctor < ApplicationRecord
  MIN_DOCTORS_ON_DUTY = 1

  scope :on_duty, -> { where(on_duty: true) }

  def check_in!
    update!(on_duty: true)
  end

  def check_out!
      unless Doctor.on_duty.count - 1 >= MIN_DOCTORS_ON_DUTY
        raise "At least #{MIN_DOCTORS_ON_DUTY} doctor(s) must be on duty"
      end

      update!(on_duty: false)
  end
end
```

To simulate concurrent access, I wrote a test script in Ruby.