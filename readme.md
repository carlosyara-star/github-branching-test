# GIT Branching Strategy

## Overview  
This branching strategy implements GitFlow, a structured approach to Git branch management designed for teams developing software with scheduled releases. GitFlow provides a robust framework for handling parallel feature development, planned releases, and critical production hotfixes without disrupting ongoing work.   
## Branch Types and Lifecycle  
### Main Branches  
#### 1. `main` -> `prod` (Production Branch)  
- **Purpose**: Contains production-ready code  
- **Lifecycle**: Permanent branch, never deleted  
- **Rules**:   
  - Only receives merges from `release/*` and `hotfix/*` branches  
  - Every commit represents a production release  
  - Always deployable  
  - Protected with branch protection rules  
   
#### 2. `develop` -> `dev` (Integration Branch)  
- **Purpose**: Integration branch for features, serves as staging for next release  
- **Lifecycle**: Permanent branch, never deleted  
- **Rules**:  
  - Receives merges from `feature/*` branches  
  - Source for creating `release/*` branches  
  - Reflects latest development changes  
  - Should be stable enough for QA testing  
   
### Supporting Branches  
   
#### 3. `feature/[Ticket-ID]` (Feature Branches)  
- **Purpose**: Develop new features  
- **Lifecycle**: Temporary - created from `develop`, merged back to `develop`, then deleted  
- **Naming**: `feature/[Ticket-ID]`  
- **Commit**:  feature: [Ticket-ID] - [Ticket-Title]  
- **Rules**:  
  - Always branch from `develop`  
  - Always merge back to `develop`  
  - Never interact directly with `main`  
   
#### 4. `release/*` (Release Branches)  
- **Purpose**: Prepare new production releases  
- **Lifecycle**: Temporary - created from `develop`, merged to both `main` and `develop`, then deleted  
- **Naming**: `release/v1.[Sprint/Release].[#]`  
- **Commit**:  Release v1.[Sprint/Release].[#]  
- **Rules**:  
  - Only bug fixes, documentation, and release-oriented tasks allowed  
  - When ready, merge to `main` and back-merge to `develop`  
   
#### 5. `hotfix/*` (Hotfix Branches)  
- **Purpose**: Fix critical production issues  
- **Lifecycle**: Temporary - created from `main`, merged to both `main` and `develop`, then deleted  
- **Naming**: `hotfix/v1.[Sprint/Release].[#]`  
- **Commit**:  hotfix: [Ticket-ID] - [Ticket-Title]  
- **Rules**:  
  - Only critical bug fixes  
  - Merge to `main` for immediate deployment  
  - Merge to `develop` to include fix in future releases  
   
## Detailed Workflows  
   
### Feature Development Workflow  
   
```bash  
# Start new feature  
git checkout develop  
git pull origin develop  
git checkout -b feature/2411  
   
# Develop feature (multiple commits)  
git add .  
git commit -m "feature: [2411] - Search Pages: Ignore leading/trailing white space"  
git push origin feature/2411  
   
# Before merging: sync with latest develop  
git checkout develop  
git pull origin develop  
git checkout feature/2411  
git rebase develop  
   
# Create Pull Request to develop  
# After approval and CI passes, merge via GitHub  
```  
### Release Workflow  
  
```bash  
# Start release branch. 
git checkout develop  
git pull origin develop  
git checkout -b release/v1.37.1  
   
# Release preparations.  
git commit -m "Release version 1.37.1"  
git push origin release/v1.37.1  
   
# After testing and fixes: Merge to main (PR) 
git checkout main  
git pull origin main  
git merge --no-ff release/v1.37.1  
   
# Merge back to develop (PR) 
git checkout develop  
git merge --no-ff release/v1.37.1  
git push origin develop  
   
# Delete release branch  
git branch -d release/v1.37.1  
git push origin --delete release/v1.37.1  
```  
   
### Hotfix Workflow  
   
```bash  
# Start hotfix from main  
git checkout main  
git pull origin main  
git checkout -b hotfix/1.37.2  
   
# Fix the critical issue  
git commit -m "Hotfix: [2412] - Fix critical security vulnerability"  
git push origin hotfix/1.37.2  
   
# Merge to main (PR) 
git checkout main  
git merge hotfix/1.37.2  
  

   
# Merge to develop (PR) 
git checkout develop  
git merge --no-ff hotfix/1.37.2  
git push origin develop  
   
# If release branch exists, merge there too  
git checkout release/1.v38.0  # if exists  
git merge --no-ff hotfix/1.37.2  
git push origin release/1.v38.0 
   
# Delete hotfix branch  
git branch -d hotfix/1.37.2  

git push origin --delete hotfix/1.37.2  

```  


Changes performed for feature/1231