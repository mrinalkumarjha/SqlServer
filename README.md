# SqlServer

# Transaction: 
              Transaction help us to group some task in a block so that either all are successful or all are revert back. ex: suppose you have customer and address table in backend and from front end user enter its detail and address both so you will be needing like either both customer and customer address insert should success or rolled back.
              So we will put both insert statement in one transaction block so that both insert get either successful or rolled back. this process is called transaction.
              
              @@ERROR : it helps us to determine error in execution.
              
              EX:
              -- TRANSACTION

            begin tran
            insert into customer(custid,name) values(1,'mrinal')
            DECLARE @custid int
            SET @custid = SCOPE_IDENTITY()
            insert into [address] ([address], custid) values('noida',@custid)

            if(@@ERROR > 0) -- @@ERROR is global variable given by sqlserver
            BEGIN
              rollback tran  -- here transaction will be rolled back
            END
            ELSE
            BEGIN
              commit tran -- otherwise transaction will be commited
            END
            
# Nested Transaction: transaction inside transaction.
            when we use nested transaction only outer commit will physically commit the transaction. @@trancount will hold number of active transaction .
            we also use check points ( save tran l1 ) while using nested transaction. we use this checkoint to save and rollback to a logical point if needed.
             
                       -- NESTED TRAN
          begin tran t1
          insert into customer(custid,name) values(1,'mrinal0')
          insert into customer(custid,name) values(1,'mrinal1')
          save tran l1 -- this is logical checkpoint
            begin tran t2
              insert into customer(custid,name) values(1,'mrinal2')
              insert into customer(custid,name) values(1,'mrinal3')
              save tran l2
              PRINT  @@trancount
            commit tran t2  -- this does not commit physically
            PRINT  @@trancount
            rollback tran l1  
          commit tran t1 -- only outer commit will physically commit
          PRINT  @@trancount

# Concurrency : 
      concurrency are the situation where two user tries to access same information. so we donot want any inconsitent result or abnormal behaviour. in below example in one query window if we run transaction code and in second query window if we try to access that 
      particular row it will be lock by default. by default row level locks are available in sql server.
      
    -- Concurrency
    -- in below example within 10 sec if someone try to get this info he will get some abnormal result. 
    -- so we want here not to show uncommited data to end user. we want to show only commited data. so we want here to 
    -- lock the record with id 22 exclusively

    -- in first query window
    begin tran
      update customer set name = 'karan1' where id = 22
      waitfor delay '00:00:10' -- delay for 10 sec
      update customer set name = 'mk2' where id = 22
    rollback tran

    -- in second query window. we will see only commited data here
    select * from customer where id = 22  -- it will be locked. 
    select * from customer where id = 23  -- this will show record as only row with id 22 wil be locked
    
    -- To read uncommited mode data also. means ok to see volatility of data with NOLOCK
    select * from customer with (nolock) where id = 22
    
# How to check locks : 
    Following are command to see locks. Mode = X is exclusive lock, type = key means Row level lock. spid -s lock no. 

    exec sp_lock
    exec sp_who  lock_no (lock_no is obtained by executing sp_lock)
    
# Deadlock :
    Deadlock is the situation where two transaction block progress of each other.
    
    -- tran 1
      begin tran
	    update customer set name = 'karan1' where id = 22
	    waitfor delay '00:00:20' -- delay for 10 sec
	    update customer set name = 'mk2' where id = 23
      rollback tran

    -- tran 2
        
      begin tran
        update customer set name = 'karan1' where id = 23
        waitfor delay '00:00:20' -- delay for 10 sec
        update customer set name = 'mk2' where id = 22
      rollback tran
    
    execute tran1 first and then tran2. in tran 1 it has execlusive access to rec 22 and in tran 2 it has exclusive access to rec 23.
    after 20 sec tran 1 tries to get exclusive access to rec 23 but it can be done as tran 2 already has exclusive access of rec 23.
    so this is scenario of deadlock. 
    
    error comes : Transaction (Process ID 53) was deadlocked on lock resources with another process and has been chosen as the deadlock victim. Rerun the transaction.
    
    in this case sql server takes call and kill one transaction.
    
# How to prevent Deadlock :
    
    
