## step 1 : Create Classes

### Rule Class
```cs
public class Rule
{
    public Rule(string parameter, dynamic criteria, string message)
    {
        Parameter = parameter;
        Criteria = criteria;
        Message = message;
    }

    public string Parameter { get; set; }
    public dynamic Criteria { get; set; }
    public string Message { get; set; }
    public bool IsBroken => !Criteria;
}
 ```
 
 ### ModelValidation static class
 ```cs
public static class ModelValidation  
{
    public static void Validate(this object model, Rule[] Rules, Xeption Exception)
    {
        foreach (var rule in Rules.Where(rule => rule.IsBroken))
        {
            Exception.UpsertDataList(key: rule.Parameter, value: rule.Message);
        }

        Exception.ThrowIfContainsErrors();
    }
}
 ```
 
 ## Step 2 : StudentService
 ### 2.1 Redifine IsValid and IsNotSame
 the validation method must define when the model is Valid otherwize the rule is broken
  ```cs
private static dynamic IsValid(string text) => !string.IsNullOrWhiteSpace(text);
private static dynamic IsValid(Guid id) => id != Guid.Empty;
private static dynamic IsTheSame(Guid firstId, Guid secondId, string secondIdName) => firstId == secondId;
 ```
 
 ## FinalStep : Validate the Model
 ```cs
 private void ValidateStudentOnAdd(Student student)
{
    Rule[] rules = new Rule[]{
        new Rule(
            parameter: nameof(Student.Id),
            criteria: IsValid(student.Id),
            message: "Id cannot be null, empty or white space"),
        new Rule(
            parameter: nameof(Student.CreatedBy),
            criteria: IsValid(student.CreatedBy),
            message: "CreatedBy cannot be null"),
        new Rule(
            parameter: nameof(Student.UpdatedBy),
            criteria: IsValid(student.UpdatedBy),
            message: "UpdatedBy cannot be null"),
        new Rule(
            parameter: nameof(Student.FirstName),
            criteria: IsValid(student.FirstName),
            message: "FirstName cannot be null, empty or white space"),
        new Rule(
            parameter: nameof(Student.LastName),
            criteria: IsValid(student.LastName),
            message: "LastName cannot be null, empty or white space"),
        new Rule(
            parameter: nameof(Student.CreatedBy),
            criteria: IsNotSame(
                firstId: student.CreatedBy,
                secondId: student.UpdatedBy,
                secondIdName: nameof(Student.UpdatedBy)),
            message: "Id cannot be null, empty or white space")
    };
    student.Validate(rules, new InvalidStudentException());
}
        
  ```
