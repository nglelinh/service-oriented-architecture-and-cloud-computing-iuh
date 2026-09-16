---
layout: post
title: 06 Introduction to Spark Ecosystem
chapter: '06'
order: 1
owner: Nguyen Le Linh
lang: en
categories:
- chapter06
---

This chapter moves beyond the core Spark engine to explore the powerful libraries built on top of it: Spark SQL for structured data, Spark Streaming for real-time processing, and MLlib for machine learning.

## Learning Objectives

- **Spark SQL**: Query structured data using SQL and DataFrames
- **Spark Streaming**: Process real-time data streams with fault tolerance
- **MLlib**: Build and deploy scalable machine learning models
- **GraphX**: (Overview) Analyze graph-structured data

## The Unified Engine

The true power of Spark lies in its unified stack. You can load data using Spark SQL, train a model using MLlib, and apply that model to a real-time stream using Spark Streaming—all within the same application.

## Optional application lesson

Required theory in this chapter is unchanged. For event-driven features, serving-adjacent ML, and stream-vs-request clocks (2022–2026), see [06-05 Real-Time, Event-Driven, and Serving-Adjacent Spark Apps]({% multilang_post_url contents/chapter06/21-01-01-06_05_Realtime_Event_Driven_ML %}).
