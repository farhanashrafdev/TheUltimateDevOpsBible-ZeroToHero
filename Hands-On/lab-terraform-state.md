# Lab: Terraform State Surgery

## 🎯 Scenario
You deleted a resource manually in AWS console, but Terraform still thinks it exists. Or you moved a resource to a module. Fix the state.

## 🛠️ Setup
1. Create a resource with Terraform (e.g., S3 bucket).
2. Manually delete it in AWS Console.
3. Run `terraform plan`.
   - Output: "Plan: 1 to add". (Terraform wants to recreate it).

## 🕵️ Resolution Steps

### Scenario A: Resource Deleted Manually
**Goal**: Tell Terraform it's gone.
```bash
terraform refresh
```
**Result**: Terraform updates state to match reality.

### Scenario B: Rename/Move Resource
**Goal**: Move `aws_s3_bucket.b` to `module.storage.aws_s3_bucket.b` without destroying it.

1. **Don't** just change the code (Terraform will destroy and recreate).
2. **Use `moved` block** (Terraform 1.1+):
```hcl
moved {
  from = aws_s3_bucket.b
  to   = module.storage.aws_s3_bucket.b
}
```
3. **Or use `state mv`**:
```bash
terraform state mv aws_s3_bucket.b module.storage.aws_s3_bucket.b
```

### Scenario C: Import Existing Resource
**Goal**: Bring an existing bucket under Terraform control.
1. Write code for resource.
2. Run import:
```bash
terraform import aws_s3_bucket.b my-existing-bucket-name
```

## ✅ Success Criteria
- [ ] `terraform plan` shows "No changes".
- [ ] Resources were NOT destroyed/recreated.
