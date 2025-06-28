# Multipage-CRUD
A step-by-step guide to building Create, Read, Update, and Delete (CRUD) functions using separate pages — one view for the table, one for editing, and one for adding new data
 
<br>  
<br> 
          
     
### 1. Create Model 

```csharp
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using System.Diagnostics.CodeAnalysis;
using System.Runtime.InteropServices;

namespace Asp.NetCore_MVC_Practice.Models
{
    public class DbTable
    {
        [Key]
        public int Id { get; set; }

        [Required]

        [MaxLength(20, ErrorMessage = "Name must be 20 characters only")]
        [DisplayName("Name")]
        public string? Name { get; set; } 


        [MaxLength(20,ErrorMessage = "Lastname must be 20 characters only")]
        [DisplayName("Lastname")]
        public string? Lastname { get; set; }


        [Range(1,100,ErrorMessage = "To old must be not required")]
        [DisplayName("Age")]
        public int? age { get; set; }

       
    }
}


```


<br>


### 2. Create  VIEW FOR TABLE,    VIEW FOR EDIT,    VIEW FOR Add Data

1. View for table 



```cshtml
@{
    ViewData["Title"] = "Student Info";
}
@model List<DbTable>

    <table class="table table-striped">
        <thead class="thead-dark">
            <tr>
                <th scope="col">Id</th>
                <th scope="col">Name</th>
                <th scope="col">Lastname</th>
                <th scope="col">Age</th>
                <th scope="col">Edit</th>
                <th scope="col">Delete</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var info in Model)
            {
                <tr>
                    <td>@info.Id</td>
                    <td>@info.Name</td>
                    <td>@info.Lastname</td>
                    <td>@info.age</td>
                    <td> <a asp-controller="StudentInfo" asp-action="Edit" asp-route-id="@info.Id" class="btn btn-warning btn-sm">Edit</a> </td>
                    <td>
                        <form asp-controller="StudentInfo" asp-action="Delete" asp-route-id="@info.Id" method="post" style="display:inline;">
                            <button type="submit" class="btn btn-danger btn-sm">Delete</button>
                        </form>
                    </td>
                </tr>
            }
        </tbody>
    </table>
</div>

```



<br>

2. View for  Add data



```cshtml
@model DbTable

<form method="post" asp-controller="StudentInfo" asp-action="add">
    <div asp-validation-summary="All" class="text-danger"></div>

    <div class="border p-4 mt-4 rounded">
        <div class="row pb-3">
            <h2 class="text-primary">Student Info</h2>
        </div>

        <div class="mb-3">
            <label asp-for="Name" class="form-label">Name</label>
            <input asp-for="Name" class="form-control" />
            <span asp-validation-for="Name" class="text-danger"></span>
        </div>

        <div class="mb-3">
            <label asp-for="Lastname" class="form-label">Lastname</label>
            <input asp-for="Lastname" class="form-control" />
            <span asp-validation-for="Lastname" class="text-danger"></span>
        </div>

        <div class="mb-3">
            <label asp-for="age" class="form-label">Age</label>
            <input asp-for="age" class="form-control" />
            <span asp-validation-for="age" class="text-danger"></span>
        </div>

        <div class="row">
            <div class="col-6">
                <button type="submit" class="btn btn-primary form-control">Add Student</button>
            </div>
            <div class="col-6">
                <a asp-controller="StudentInfo" asp-action="Index" class="btn btn-secondary form-control">Back</a>
            </div>
        </div>
    </div>
</form>


```





3. View for Edit



```cshtml
@model DbTable

<form method="post">

	<div asp-validation-summary="All" class="validate"></div>

	<div class="border p-3 mt-4">
		<div class="row pb-2">
			<h2 class="text-primary">Edit Student Info</h2>
		</div>
	</div>

	<div class="mb-3">
		<label>Name</label>
		<input asp-for="Name" class="form-control" />
		<span asp-validation-for="Name" class="text-danger"></span>
	</div>

	<div class="mb-3">
		<label>Lastname</label>
		<input asp-for="Lastname" class="form-control" />
		<span asp-validation-for="Lastname" class="text-danger"></span>
	</div>

	<div class="mb-3">
		<label>Age</label>
		<input asp-for="age" class="form-control" />
		<span asp-validation-for="age" class="text-danger"></span>
	</div>

	<div class="row">
		<div class="col-6">
			<button type="submit" class="btn btn-primary form-control">Update Student</button>
		</div>
	</div>

</form>


```



<br>
<br>


4. If you want to add a search feature, here are some options you can try.
   
Place this code above the table section to ensure it works properly.


```cshtml

<form asp-controller="StudentInfo" asp-action="Search" method="post">
	<input type="text" name="Searchstring"/>
	<input type="submit" value="Search" />
</form>


```


<br>
<br>
<br>


# After creating the views, the next step is to create the controller that will handle their logic.

<br>

1. Controller logic for Table View

```csharp

        //display data from table in html
        public IActionResult Index()
        {
            List<Models.DbTable> studentInfoList = _db.StudentInfo.ToList();
            return View(studentInfoList);
        }
```


<br>


2. Controller logic for Input or Add data

```csharp


        //add record to the database
        [HttpPost]
        public IActionResult add(Models.DbTable obj)
        {
            _db.StudentInfo.Add(obj);
            _db.SaveChanges();
            return RedirectToAction("index");
        }
       
```


<br>


3. Controller logic for Edit

```csharp


      
        //Retrieve the ID from the route so you can load the correct data into the edit form.
        public IActionResult Edit(int id)
        {
            var GetEdit = _db.StudentInfo.Find(id);
            return View(GetEdit);
        }


        //Update the record in the database.
        [HttpPost]
        public IActionResult Edit(Models.DbTable studentData)
        {

            var edit = _db.StudentInfo.Find(studentData.Id);


            if (edit != null)
            {
                edit.Name = studentData.Name;
                edit.Lastname = studentData.Lastname;
                edit.age = studentData.age;

                _db.SaveChanges();
            }
            return RedirectToAction("index");
        }

       
```


<br>


4. Controller for Delete data

  ```csharp

     [HttpPost]
     public IActionResult Delete(int id)
     {
         var record =  _db.StudentInfo.Find(id);
         if(record == null)
         {
             return NotFound();
         }
         _db.StudentInfo.Remove(record);
         _db.SaveChanges();

         return RedirectToAction("index");
     }

       
```

<br>

5. Controller for Search Data

  ```csharp

    
        [HttpPost]
        public async Task<IActionResult> Search(string Searchstring)
        {
            if (_db.StudentInfo == null)
            {
                return Problem("");
            }
            var names = from m in _db.StudentInfo select m;

            if (!String.IsNullOrEmpty(Searchstring))
            {
                names = names.Where(s => s.Name!.ToUpper().Contains(Searchstring.ToUpper()));
            }

            return View("index",await names.ToListAsync());

        }

       
``` 





