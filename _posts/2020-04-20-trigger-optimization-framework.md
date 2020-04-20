---
title: Trigger Optimization Framework in Salesforce
layout: blog
category: [Salesforce]
excerpt: In this blog post, we will do a deep dive into a comprehensive Trigger Framework that we can adopt in Salesforce Apex programming.
comments: true
---

Firstly, I need to acknowledge the valuable inputs that [Adam Purkiss](https://twitter.com/apurkiss){:target="\_blank"} and [Hari Krishnan](https://krishhari.wordpress.com/about/){:target="\_blank"} have contributed in the Salesforce community on apex trigger frameworks and its management.

> **Disclaimer**: Many organizations and people in the community have their own version of Trigger Framework. That is totally fine and this blog is not to prove them wrong.

In this blog, I have tried to extend the trigger framework to be more configurable in nature. If you have any suggestions/concerns, please feel free to comment on this blog post.

#### What you need to know before you read this blog

1. [Basic understanding on Salesforce Apex Triggers](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers.htm){:target="\_blank"}
2. [Understanding of various Trigger Context Variables](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_context_variables.htm){:target="\_blank"}
3. [Order of Execution in Salesforce](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_triggers_order_of_execution.htm){:target="\_blank"}

#### Current Problems with Apex Triggers

1. **Many triggers for one object** – Having more than 1 triggger per object causes the prediction of Execution Order to be difficult.
2. **Business Logic scattered in triggers & classes** – Business Logic gets scattered in some triggers, handler classes and helper classes. Thus, it causes maintenance problems.
3. **Re-entry of trigger logic** – Difficult to manage & maintain these as many developers have their own approach of managing re-entries.
4. **Number of SOQL and DML statements executed** in a trigger context somtimes goes out of control
5. **Difficult to de-couple logic** – There needs to be lot of impact analysis due to scattered code before you remove or pull some logic out.
6. **Not enough logs generated** – Logging is not consistent from triggers to classes. Usually, developers have to write logging statements here and there.

#### Benefits of a Trigger Framework

1. Makes the **Triggers more streamlined** and provides a precise predictability in order of execution
2. It provides **separate code paths for new & re-entrant logic**. Thus making the management of logic simpler and impact-free.
3. Provides a **Centralized SOQL and DML management** by which SOQL and DML statements can be managed in a central location rather than making changes everywhere.
4. Facilitates **Logging at the framework level**. Thus, making the developers focus on writing the logic efficiently and not worry about exception handling around each line they write.
5. Provides **ability to switch ON/OFF triggers based on configurations**. This helps us when issues occur in production and we have to troubleshoot it without deploying code.

The trigger framework orchestrates its execution as shown below.
![Trigger Execution Pipeline](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/trigger-execution-pipeline.png)

##### Explanation of the methods in the trigger pipeline

1. `bulkBefore`- This point is an ideal location where we can do anything before the actual before trigger logic starts to work. It is a good choice to centralize all the data using SOQL statements and keep the data ready in a list/map so that the before trigger handlers can get/set properties of the already queried records.
2. `applyDefault` - Particularly for beforeInsert, it's best to ensure that the default values of the incoming record be set based on any business expectation. This will ensure that the values of the record is always consistent in terms of business expectation. Note that this only applies to the beforeInsert pipeline.
3. `beforeInsert` - This is the point where the actual beforeInsert logic needs to be plugged in [with the correct order of operations that business expects]
4. `beforeUpdate` - This is the point where the actual beforeUpdate logic needs to be plugged in [with the correct order of operations that business expects]
5. `beforeDelete` - This is the point where the actual beforeDelete logic needs to be plugged in [with the correct order of operations that business expects]
6. `bulkAfter` - This point is an ideal location where we can do anything just before the actual after trigger logic starts to work. It is a good choice to centralize all the data using SOQL statements and keep the data ready in a list/map so that the after trigger handlers can get/set properties of the already queried records.
7. `onValidate` - Typically, on any after triggers, it is always good to validate the consistency of the records and ensure that errors are thrown back if validation fails. This `onValidate()` saves from any potential after triggers throwing an exception just because a record is not consistent.
8. `afterInsert` - This is the point where the actual afterInsert logic needs to be plugged in [with the correct order of operations that business expects]
9. `afterUpdate` - This is the point where the actual afterUpdate logic needs to be plugged in [with the correct order of operations that business expects]
10. `afterDelete` - This is the point where the actual afterDelete logic needs to be plugged in [with the correct order of operations that business expects]
11. `afterUnDelete` - This is the point where the actual afterUnDelete logic needs to be plugged in [with the correct order of operations that business expects]. UnDelete happens when a record lying in the Recycle Bin is restored back to the object.

In a nutshell, what did we try to streamline using this trigger pipeline?

- Before Trigger Execution (4th point in the order of execution)
- After Trigger Execution (8th point in the order of execution)

  ![Order of Execution](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/order-of-execution.png)

Every trigger execution context can now be separated from the pipeline by having its own handlers. For example, `beforeInsert` for Lead can be separated into a class called **LeadBeforeInsertHandler**.
Why we need to separate it? It ensures **separation of concerns** and that we are not mixing handler responsibilities in trigger pipeline orchestration.

Let's now look at how Handler execution can be generalized.
![Handler Pipeline](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/handler-execution-pipeline.png)

##### Explanation of steps in the Handler pipeline

1. **Picking the Handler Instance** - Based on the incoming object type, we need to pick and instantiate the correct handler class
2. **mainEntry** - This is the method that needs to execute if the trigger control flow is being invoked freshly in a context and it is not as a result of any re-entry.
3. **inProgressEntry** - This is the method that needs to execute in case the trigger control re-enters due to any DML's executed within the trigger lifetime.
4. **Centralized DML** - After a trigger handler is executed, it would have set some properties of the records and this is the point where we can centrally execute DML statements on the collection of records.

#### ITriggerDispatcher

Interface that contains the contracts of the main trigger pipeline

```java
    public interface ITriggerDispatcher
    {
        void applyDefaults(TriggerParameter tp);
        void bulkBefore(TriggerParameter tp);
        void beforeInsert(TriggerParameter tp);
        void beforeUpdate(TriggerParameter tp);
        void beforeDelete(TriggerParameter tp);

        void bulkAfter(TriggerParameter tp);
        void onValidate(TriggerParameter tp);
        void afterInsert(TriggerParameter tp);
        void afterUpdate(TriggerParameter tp);
        void afterDelete(TriggerParameter tp);
        void afterUnDelete(TriggerParameter tp);

        void andFinally();
    }
```

#### ITriggerHandler

Interface that contains the contracts of the trigger context execution

```java
    public interface ITriggerHandler
    {
        void mainEntry(TriggerParameter tp);
        void inProgressEntry(TriggerParameter tp);

        void insertObjects();
        void updateObjects();
        void deleteObjects();
    }
```

#### TriggerParameter

Encapsulated form of the trigger context variables

```java
    public class TriggerParameter
    {
        public Enum TriggerEvent { beforeInsert, beforeUpdate, beforeDelete, afterInsert, afterUpdate, afterDelete, afterUndelete }
        public TriggerEvent tEvent;

        public List<SObject> oldList { get; private set; }
        public List<SObject> newList { get; private set; }
        public Map<Id, SObject> oldMap { get; private set; }
        public Map<Id, SObject> newMap { get; private set; }
        public String triggerObject { get; private set; }
        public Boolean isExecuting { get; private set; }

        public TriggerParameter(List<SObject> olist, List<SObject> nlist, Map<Id, SObject> omap, Map<Id, SObject> nmap, Boolean ib, Boolean ia, Boolean id, Boolean ii, Boolean iu, Boolean iud, Boolean ie) {
            this.oldList = olist;
            this.newList = nlist;
            this.oldMap = omap;
            this.newMap = nmap;
            this.triggerObject = UtilityClass.getSObjectTypeName((this.oldList != null && this.oldList.size() > 0) ? this.oldList[0] : this.newList[0]);
            if (ib & ii) tEvent = TriggerEvent.beforeInsert;
            else if (ib && iu) tEvent = TriggerEvent.beforeUpdate;
            else if (ib && id) tEvent = TriggerEvent.beforeDelete;
            else if (ia && ii) tEvent = TriggerEvent.afterInsert;
            else if (ia && iu) tEvent = TriggerEvent.afterUpdate;
            else if (ia && id) tEvent = TriggerEvent.afterDelete;
            else if (ia && iud) tEvent = TriggerEvent.afterUndelete;
            isExecuting = ie;
        }
    }
```

#### UtilityClass

Helper Class that contains methods commonly used in the framework

- Note that we are using a Custom Metadata Type named `Salesforce_Object__mdt` that contains the dispatcher class name registered for the object
- Similarly, we are using another Custom Metadata Type named `Trigger_Handler__mdt` that contains the handler class name registered for the object and its active state
- Observe that we are using `Type` system in Apex language to instantiate the dispatcher and handler instances of the incoming object type that is picked from configurations
- If a specific handler is set as Inactive in the configuration, all other handlers would work except for the disabled one. The way we do it is by invoking an EmptyHandler when the main handler is set as Inactive.

```java
    public with sharing class UtilityClass {
        public static String getSObjectTypeName(SObject so) {
            return so.getSObjectType().getDescribe().getName();
        }

        public static ITriggerHandler getHandler(TriggerParameter tp) {
            List<Trigger_Handler__mdt> handlerConfig = [select DeveloperName, IsActive__c from Trigger_Handler__mdt where Salesforce_Object__r.DeveloperName=:tp.triggerObject and Trigger_Context__c=:tp.tEvent.name() LIMIT 1];
            if(handlerConfig.size()==0)
            {
                throw new TriggerException('Handler Configuration for '+tp.triggerObject+ ' - '+ tp.tEvent.name() +' is not defined');
            }
            else
            {
                try
                {
                    string handlerName = 'EmptyHandler';
                    if(handlerConfig[0].IsActive__c)
                        handlerName = handlerConfig[0].DeveloperName;
                    Type classType = Type.forName(handlerName);
                    ITriggerHandler handlerInstance = (ITriggerHandler)classType.newInstance();
                    system.debug('# Handler in charge is: '+ handlerName);
                    return handlerInstance;
                }
                catch (Exception e)
                {
                    throw new TriggerException(handlerConfig[0].DeveloperName+' class doesnt seem to exist. Please check with development team.');
                }
            }
        }

        public static ITriggerDispatcher getDispatcherObject(string sObjectName){
            List<Salesforce_Object__mdt> dispatcherConfig = [select Dispatcher_Class_Name__c from Salesforce_Object__mdt where DeveloperName=:sObjectName LIMIT 1];
            if(dispatcherConfig.size()==0)
            {
                throw new TriggerException('Dispatcher Configuration for '+sObjectName+' is not defined');
            }
            else
            {
                try
                {
                    Type classType = Type.forName(dispatcherConfig[0].Dispatcher_Class_Name__c);
                    ITriggerDispatcher dispatcherInstance = (ITriggerDispatcher)classType.newInstance();
                    return dispatcherInstance;
                }
                catch (Exception e)
                {
                    throw new TriggerException(dispatcherConfig[0].Dispatcher_Class_Name__c+' class doesnt seem to exist. Please check with development team.');
                }

            }
        }
    }
```

#### Custom Trigger Exception

Custom Trigger Exception used in the framework

```java
    public class TriggerException extends Exception {

    }
```

#### Trigger Dispatcher Base Class

Implements the ITriggerDispatcher and houses the default implementations of how to invoke the handlers in what sequence

```java
    public abstract class TriggerDispatcherBase implements ITriggerDispatcher
    {
        private static ITriggerHandler beforeInserthandler;
        private static ITriggerHandler beforeUpdatehandler;
        private static ITriggerHandler beforeDeleteHandler;
        private static ITriggerHandler afterInserthandler;
        private static ITriggerHandler afterUpdatehandler;
        private static ITriggerHandler afterDeleteHandler;
        private static ITriggerHandler afterUndeleteHandler;

        private static boolean isBeforeInsertProcessing = false;
        private static boolean isBeforeUpdateProcessing = false;
        private static boolean isBeforeDeleteProcessing = false;
        private static boolean isAfterInsertProcessing = false;
        private static boolean isAfterUpdateProcessing = false;
        private static boolean isAfterDeleteProcessing = false;
        private static boolean isAfterUnDeleteProcessing = false;

        public abstract void bulkBefore(TriggerParameter tp);
        public abstract void applyDefaults(TriggerParameter tp);
        public abstract void bulkAfter(TriggerParameter tp);
        public abstract void onValidate(TriggerParameter tp);

        public void beforeInsert(TriggerParameter tp) {
            system.debug('# Finding the Before Insert Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isBeforeInsertProcessing){
                    isBeforeInsertProcessing = true;
                    system.debug('# Running Before Insert Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.beforeInsert);
                    isBeforeInsertProcessing = false;
                }
                else {
                    system.debug('# Running Before Insert Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.beforeInsert);
                }
            }
        }
        public void beforeUpdate(TriggerParameter tp) {
            system.debug('# Finding the Before Update Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isBeforeUpdateProcessing){
                    isBeforeUpdateProcessing = true;
                    system.debug('# Running Before Update Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.beforeUpdate);
                    isBeforeUpdateProcessing = false;
                }
                else {
                    system.debug('# Running Before Update Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.beforeUpdate);
                }
            }
        }
        public void beforeDelete(TriggerParameter tp) {
            system.debug('# Finding the Before Delete Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isBeforeDeleteProcessing){
                    isBeforeDeleteProcessing = true;
                    system.debug('# Running Before Delete Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.beforeDelete);
                    isBeforeDeleteProcessing = false;
                }
                else {
                    system.debug('# Running Before Delete Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.beforeDelete);
                }
            }
        }
        public void afterInsert(TriggerParameter tp) {
            system.debug('# Finding the After Insert Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isAfterInsertProcessing){
                    isAfterInsertProcessing = true;
                    system.debug('# Running After Insert Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.afterInsert);
                    isAfterInsertProcessing = false;
                }
                else {
                    system.debug('# Running After Insert Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.afterInsert);
                }
            }
        }
        public void afterUpdate(TriggerParameter tp) {
            system.debug('# Finding the After Update Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isAfterUpdateProcessing){
                    isAfterUpdateProcessing = true;
                    system.debug('# Running After Update Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.afterUpdate);
                    isAfterUpdateProcessing = false;
                }
                else {
                    system.debug('# Running After Update Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.afterUpdate);
                }
            }
        }
        public void afterDelete(TriggerParameter tp) {
            system.debug('# Finding the After Delete Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isAfterDeleteProcessing){
                    isAfterDeleteProcessing = true;
                    system.debug('# Running After Delete Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.afterDelete);
                    isAfterDeleteProcessing = false;
                }
                else {
                    system.debug('# Running After Delete Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.afterDelete);
                }
            }
        }
        public void afterUnDelete(TriggerParameter tp) {
            system.debug('# Finding the After UnDelete Trigger Handler');
            ITriggerHandler handler = UtilityClass.getHandler(tp);
            if(handler!=null){
                if(!isAfterUnDeleteProcessing){
                    isAfterUnDeleteProcessing = true;
                    system.debug('# Running After UnDelete Trigger Handler - Main Entry');
                    execute(handler, tp, TriggerParameter.TriggerEvent.afterUnDelete);
                    isAfterUnDeleteProcessing = false;
                }
                else {
                    system.debug('# Running After UnDelete Trigger Handler - In Progress Entry');
                    execute(null, tp, TriggerParameter.TriggerEvent.afterUnDelete);
                }
            }
        }
        public virtual void andFinally() {}

        public void execute(ITriggerHandler handlerInstance, TriggerParameter tp, TriggerParameter.TriggerEvent tEvent) {
            if(handlerInstance != null) {
                if(tEvent == TriggerParameter.TriggerEvent.beforeInsert)
                    beforeInsertHandler = handlerInstance;
                if(tEvent == TriggerParameter.TriggerEvent.beforeUpdate)
                    beforeUpdateHandler = handlerInstance;
                if(tEvent == TriggerParameter.TriggerEvent.beforeDelete)
                    beforeDeleteHandler = handlerInstance;
                if(tEvent == TriggerParameter.TriggerEvent.afterInsert)
                    afterInsertHandler = handlerInstance;
                if(tEvent == TriggerParameter.TriggerEvent.afterUpdate)
                    afterUpdateHandler = handlerInstance;
                if(tEvent == TriggerParameter.TriggerEvent.afterDelete)
                    afterDeleteHandler = handlerInstance;
                if(tEvent == TriggerParameter.TriggerEvent.afterUnDelete)
                    afterUndeleteHandler = handlerInstance;

                handlerInstance.mainEntry(tp);

                system.debug('# DML Insert - Main Entry');
                handlerInstance.insertObjects();
                system.debug('# DML Update - Main Entry');
                handlerInstance.updateObjects();
                system.debug('# DML Delete - Main Entry');
                handlerInstance.deleteObjects();
            }
            else {
                if(tEvent == TriggerParameter.TriggerEvent.beforeInsert)
                    beforeInsertHandler.inProgressEntry(tp);
                if(tEvent == TriggerParameter.TriggerEvent.beforeUpdate)
                    beforeUpdateHandler.inProgressEntry(tp);
                if(tEvent == TriggerParameter.TriggerEvent.beforeDelete)
                    beforeDeleteHandler.inProgressEntry(tp);
                if(tEvent == TriggerParameter.TriggerEvent.afterInsert)
                    afterInsertHandler.inProgressEntry(tp);
                if(tEvent == TriggerParameter.TriggerEvent.afterUpdate)
                    afterUpdateHandler.inProgressEntry(tp);
                if(tEvent == TriggerParameter.TriggerEvent.afterDelete)
                    afterDeleteHandler.inProgressEntry(tp);
                if(tEvent == TriggerParameter.TriggerEvent.afterUnDelete)
                    afterUndeleteHandler.inProgressEntry(tp);

                system.debug('# DML Insert - In Progress Entry');
                handlerInstance.insertObjects();
                system.debug('# DML Update - In Progress Entry');
                handlerInstance.updateObjects();
                system.debug('# DML Delete - In Progress Entry');
                handlerInstance.deleteObjects();
            }
        }
    }
```

#### Trigger Handler Base Class

Implements the ITriggerHandler interface and houses the default implementations of the handler methods

- Note here that we are using a custom object named `Apex_Debug_Log__c` to record the exceptions automatically caught in the handler execution.
- mainEntry method is `abstract`. This means that developers need to write the implementation for this method without any fail.
- inProgressEntry method is `virtual`. This means that developers may choose to override this method in their handler class in case they have a logic to wire up specifically for re-entrant code.

```java
    public abstract class TriggerHandlerBase implements ITriggerHandler
    {
        protected List<SObject> sObjectsToInsert = new List<SObject>();
        protected List<SObject> sObjectsToUpdate = new List<SObject>();
        protected List<SObject> sObjectsToDelete = new List<SObject>();

        public abstract void mainEntry(TriggerParameter tp);
        public virtual void inProgressEntry(TriggerParameter tp){}

        public virtual void insertObjects(){
            if(sObjectsToInsert.size() > 0){
                try {
                    Database.SaveResult[] results = Database.insert(sObjectsToInsert, false);
                    if(results != null)
                    {
                        List<Apex_Debug_Log__c> logs = new List<Apex_Debug_Log__c>();
                        for(Database.SaveResult result: results)
                        {
                            if(!result.isSuccess())
                            {
                                Database.Error[] errors = result.getErrors();
                                for(Database.Error error : errors)
                                {
                                    logs.add(new Apex_Debug_Log__c(
                                        Debug_Message__c = error.getStatusCode() + ' - ' + error.getMessage(),
                                        Debug_Stack_Trace__c = 'Fields that affected this error are: ' + error.getFields(),
                                        Log_Level__c = 'Error',
                                        Object_Type__c = GetObjectNameFromRecordID(result.getId()),
                                        Record_ID__c = result.getId(),
                                        Class__c = 'TriggerHandlerBase',
                                        Method__c = 'insertObjects'
                                    ));
                                }
                            }
                        }
                    }
                } catch (Exception e) {
                    throw e;
                }
            }
        }

        public virtual void updateObjects(){
            if(sObjectsToUpdate.size() > 0){
                try {
                    Database.SaveResult[] results = Database.update(sObjectsToUpdate, false);
                    if(results != null)
                    {
                        List<Apex_Debug_Log__c> logs = new List<Apex_Debug_Log__c>();
                        for(Database.SaveResult result: results)
                        {
                            if(!result.isSuccess())
                            {
                                Database.Error[] errors = result.getErrors();
                                for(Database.Error error : errors)
                                {
                                    logs.add(new Apex_Debug_Log__c(
                                        Debug_Message__c = error.getStatusCode() + ' - ' + error.getMessage(),
                                        Debug_Stack_Trace__c = 'Fields that affected this error are: ' + error.getFields(),
                                        Log_Level__c = 'Error',
                                        Object_Type__c = GetObjectNameFromRecordID(result.getId()),
                                        Record_ID__c = result.getId(),
                                        Class__c = 'TriggerHandlerBase',
                                        Method__c = 'updateObjects'
                                    ));
                                }
                            }
                        }
                    }
                } catch (Exception e) {
                    throw e;
                }
            }
        }

        public virtual void deleteObjects(){
            if(sObjectsToDelete.size() > 0){
                try {
                    Database.DeleteResult[] results = Database.delete(sObjectsToDelete, false);
                    if(results != null)
                    {
                        List<Apex_Debug_Log__c> logs = new List<Apex_Debug_Log__c>();
                        for(Database.DeleteResult result: results)
                        {
                            if(!result.isSuccess())
                            {
                                Database.Error[] errors = result.getErrors();
                                for(Database.Error error : errors)
                                {
                                    logs.add(new Apex_Debug_Log__c(
                                        Debug_Message__c = error.getStatusCode() + ' - ' + error.getMessage(),
                                        Debug_Stack_Trace__c = 'Fields that affected this error are: ' + error.getFields(),
                                        Log_Level__c = 'Error',
                                        Object_Type__c = GetObjectNameFromRecordID(result.getId()),
                                        Record_ID__c = result.getId(),
                                        Class__c = 'TriggerHandlerBase',
                                        Method__c = 'deleteObjects'
                                    ));
                                }
                            }
                        }
                    }
                } catch (Exception e) {
                    throw e;
                }
            }
        }

        private string GetObjectNameFromRecordID(Id objectId){
            return objectId.getSObjectType().getDescribe().getName();
        }
    }
```

#### TriggerAgent

This class acts like an intermediary between the actual trigger and the trigger framework. It passes the trigger context data to the framework so that dispatchers can start the execution pipeline.

```java
    public class TriggerAgent
    {
        public static void RunTrigger(Schema.SObjectType sObjectType){
            system.debug('# Entering TriggerAgent');
            string sObjectName = sObjectType.getDescribe().getName();
            system.debug('# Finding the Trigger Dispatcher');
            ITriggerDispatcher dispatcher = UtilityClass.getDispatcherObject(sObjectName);
            if (dispatcher == null)
                throw new TriggerException('No Trigger dispatcher registered for Object Type: ' + sObjectName);
            system.debug('# Starting the Trigger Execution Order');
            executeTriggerFlow(dispatcher);
        }

        private static void executeTriggerFlow(ITriggerDispatcher dispatcher){
            system.debug('# Preparing the TriggerParameter object');
            TriggerParameter tp = new TriggerParameter(Trigger.old, Trigger.new, Trigger.oldMap, Trigger.newMap, Trigger.isBefore, Trigger.isAfter, Trigger.isDelete, Trigger.isInsert, Trigger.isUpdate, Trigger.isUnDelete, Trigger.isExecuting);

            if(Trigger.isBefore){
                system.debug('# BulkBefore');
                dispatcher.bulkBefore(tp);
                if(Trigger.isDelete){
                    system.debug('# BeforeDelete');
                    dispatcher.beforeDelete(tp);
                }
                else if (Trigger.isInsert){
                    system.debug('# ApplyDefaults');
                    dispatcher.applyDefaults(tp);
                    system.debug('# BeforeInsert');
                    dispatcher.beforeInsert(tp);
                }
                else if (Trigger.isUpdate){
                    system.debug('# BeforeUpdate');
                    dispatcher.beforeUpdate(tp);
                }
            }
            else {
                system.debug('# BulkAfter');
                dispatcher.bulkAfter(tp);
                if(Trigger.isDelete){
                    system.debug('# AfterDelete');
                    dispatcher.afterDelete(tp);
                }
                else if (Trigger.isInsert){
                    system.debug('# AfterInsert - OnValidate');
                    dispatcher.onValidate(tp);
                    system.debug('# AfterInsert');
                    dispatcher.afterInsert(tp);
                }
                else if (Trigger.isUpdate){
                    system.debug('# AfterUpdate - OnValidate');
                    dispatcher.onValidate(tp);
                    system.debug('# AfterUpdate');
                    dispatcher.afterUpdate(tp);
                }
                else if (Trigger.isUnDelete){
                    system.debug('# AfterUnDelete');
                    dispatcher.afterUnDelete(tp);
                }
            }
            system.debug('# Finally');
            dispatcher.andFinally();
        }
    }
```

#### Concrete implementation of Lead Trigger Dispatcher

The actual dispatcher implementation that a developer needs to write

```java
    public class LeadTriggerDispatcher extends TriggerDispatcherBase
    {
        public override void bulkBefore(TriggerParameter tp)
        {
        }

        public override void applyDefaults(TriggerParameter tp)
        {
        }

        public override void bulkAfter(TriggerParameter tp)
        {
        }

        public override void onValidate(TriggerParameter tp)
        {
        }
    }
```

#### Concrete implementation of Lead BeforeInsert Handler

The actual beforeInsert handler implementation that a developer needs to write

```java
    public class LeadBeforeInsert extends TriggerHandlerBase
    {
        public override void mainEntry(TriggerParameter tp)
        {
            system.debug('# Executing LeadBeforeInsert logic');
        }
    }
```

#### Concrete implementation of Empty Handler

One time implementation of the Empty Handler which is used to bypass an inactive handler

```java
    public class EmptyHandler extends TriggerHandlerBase
    {
        public override void mainEntry(TriggerParameter tp)
        {
            // DO NOT WRITE LOGIC. THIS IS MEANT TO BE EMPTY.
        }
    }
```

#### Lead Trigger [single trigger]

Implementation of the apex trigger that a developer needs to write

```java
    trigger LeadTrigger on Lead (before insert, before update, before delete, after insert, after update, after delete, after undelete)
    {
        TriggerAgent.RunTrigger(Lead.sObjectType);
    }
```

#### Salesforce_Object\_\_mdt

##### Field Structure

| Field Name                 | Type     |
| -------------------------- | -------- |
| Dispatcher_Class_Name\_\_c | Text(40) |

##### Values in Salesforce_Object\_\_mdt custom metadata type

| Salesforce Object Name | Dispatcher Class Name |
| ---------------------- | --------------------- |
| Lead                   | LeadTriggerDispatcher |

#### Trigger_Handler\_\_mdt

##### Field Structure

| Field Name        | Type                                      |
| ----------------- | ----------------------------------------- |
| IsActive          | Checkbox                                  |
| Salesforce Object | Metadata Relationship (Salesforce Object) |
| Trigger Contect   | Picklist                                  |

##### Values in Trigger_Handler\_\_mdt custom metadata type

| Salesforce Object | Trigger Handler Name | Trigger Context | IsActive |
| ----------------- | -------------------- | --------------- | -------- |
| Lead              | LeadBeforeInsert     | beforeInsert    | true     |
| Lead              | LeadBeforeUpdate     | beforeUpdate    | false    |
| Lead              | LeadBeforeDelete     | beforeDelete    | false    |
| Lead              | LeadAfterInsert      | afterInsert     | false    |
| Lead              | LeadAfterUpdate      | afterUpdate     | false    |
| Lead              | LeadAfterDelete      | afterDelete     | false    |
| Lead              | LeadAfterUnDelete    | afterUndelete   | false    |

#### TEST

- Open the developer console
- Create a Lead record from the Sales app in Salesforce
- Inspect the debug log

![Debug Log](https://abhisheksubbusite.s3-ap-southeast-1.amazonaws.com/images/trigger-execution-debug.png)

> I will be improving this blog with more data on Trigger Exception Handling & Test Class Approach. Stay tuned....
