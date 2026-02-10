# **Github Basic Commands**  
**Boritrade, LLC**  
**Last Updated:** January 30, 2025  

## **Basic Github Commands**

### **Creating Branches**

1. **Create a new branch locally**:
   ```bash
   git branch <branch-name>
   ```

2. **Create and switch to a new branch**:
   ```bash
   git checkout -b <branch-name>
   ```
   (or, in Git 2.23 and later):
   ```bash
   git switch -c <branch-name>
   ```

3. **Push the new branch to a remote repository**:
   ```bash
   git push -u origin <branch-name>
   ```

---

### **Switching Branches**

1. **Switch to an existing branch locally**:
   ```bash
   git checkout <branch-name>
   ```
   (or, in Git 2.23 and later):
   ```bash
   git switch <branch-name>
   ```

2. **List all branches (local and remote)**:
   ```bash
   git branch -a
   ```

3. **List only local branches**:
   ```bash
   git branch
   ```

4. **List remote branches**:
   ```bash
   git branch -r
   ```

---

### **Deleting Branches**

#### **Local Branch**

1. **Delete a local branch**:
   ```bash
   git branch -d <branch-name>
   ```
   *(Only works if the branch has been fully merged.)*

2. **Force delete a local branch**:
   ```bash
   git branch -D <branch-name>
   ```

#### **Remote Branch**

1. **Delete a remote branch**:
   ```bash
   git push origin --delete <branch-name>
   ```

2. **Alternative way (older versions)**:
   ```bash
   git push origin :<branch-name>
   ```

---

### **Working with Remote Branches**

1. **Fetch remote branches**:
   ```bash
   git fetch
   ```

2. **Create a local branch from a remote branch**:
   ```bash
   git checkout -b <branch-name> origin/<branch-name>
   ```
   (or, in Git 2.23 and later):
   ```bash
   git switch -t origin/<branch-name>
   ```

3. **Track a remote branch**:
   ```bash
   git branch --track <branch-name> origin/<branch-name>
   ```

4. **Set upstream for a branch**:
   ```bash
   git branch --set-upstream-to=origin/<branch-name>
   ```

5. **Rename a local branch and update its remote tracking**:
   ```bash
   git branch -m <old-name> <new-name>
   git push origin -u <new-name>
   git push origin --delete <old-name>
   ```

---

### **Clean Up**

1. **Delete stale remote-tracking branches**:
   ```bash
   git fetch --prune
   ```

2. **Prune deleted remote branches**:
   ```bash
   git remote prune origin
   ```
