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

