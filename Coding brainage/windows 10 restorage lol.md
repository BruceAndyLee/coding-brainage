---
Status: Completed
tags:
- chore
- HOW
---
MrBasher said:

> I wonder if someone could shed some light on a System Restore problem I am having, when I try to a restore it goes to my username and asks for a password, I have tried putting in my MS account password and my Windows logon pin but get know joy. I have never setup a password for this, can anyone shed any light for me please...

When you boot to Advanced Startup using the PC's own recovery environment then it requires that you are verified as an admin user against the local account database. You are asked to provide the password of a **local account** that is an administrator. Your problem is that you probably have had an MS account from the start and have no way to know what (if any) is the **local** password for your account.

Two ways around this, the first is to boot from the install media or a recovery drive as has already been suggested. The second would be to create a new local account, set a password for it and make it an administrator. The next time you boot to the PC's recovery environment it will ask you which administrator account you want to use. Choose the new local account for which you know the password.

[[5440addlocala]]

Apart from anything else, it's always a good idea to have another local admin account as a way in should your main account ever get corrupted or you get yourself locked out.