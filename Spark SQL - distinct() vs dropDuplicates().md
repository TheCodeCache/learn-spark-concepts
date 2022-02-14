# distinct() 

![image](https://user-images.githubusercontent.com/26399543/153939718-5828dfa7-d768-49d4-9ec1-e5bc80079175.png)  

**`distinct()`** does not accept any arguments which means that,  
we cannot select which columns need to be taken into account when dropping the duplicates.  
This means that the following command will drop the duplicate records taking into account all the columns of the dataframe:  

![image](https://user-images.githubusercontent.com/26399543/153939837-d064e498-dc66-4f66-912b-9819c3d77307.png)  

Now in case we want to drop the duplicates considering ONLY `id` and `name`  
we'd have to run a `select()` prior to `distinct()`. For example,  

![image](https://user-images.githubusercontent.com/26399543/153939976-6f7e1437-0538-4db2-b820-099de4d0ae99.png)  

But if we want to drop the duplicates only over a subset of columns like above but keep ALL the columns,  
then `distinct()` would not help.  

# dropDuplicates()

**`dropDuplicates()`** will drop the duplicates detected over the provided set of columns,  
but it will also return all the columns appearing in the original dataframe.  

![image](https://user-images.githubusercontent.com/26399543/153940256-a8d47583-e409-4e81-a386-c2b76cec06e3.png)  

`dropDuplicates()` is thus more suitable when we want to drop duplicates over a selected subset of columns,  
but also want to keep all the columns:  

![image](https://user-images.githubusercontent.com/26399543/153940331-94b8140d-62b5-433b-b968-719022c6c08d.png)  


**In other words:**  

The main difference is the consideration of the subset of columns which is great!  
When using distinct we need a prior `.select` to select the columns on which we want to apply the duplication  
and the returned Dataframe contains only these selected columns  
while `dropDuplicates(colNames)` will return all the columns of the initial dataframe  
after removing duplicated rows as per the columns.  

**Reference:**  
1. https://stackoverflow.com/a/64431686/6842300
2. https://stackoverflow.com/a/53546115/6842300

