# Copier template inheritance via task and migrations demo: Child Template

Parent template can be found [here](https://github.com/francesco086/copier-inheritance-via-task-and-migration-demo-parent).

This child template has 3 versions: `v1.0.0`, `v1.1.0`, and `v2.0.0`.

User can experiment using `copier copy` and `copier update` commands on all these versions, and it should work.

I tried these two experiments, and they worked as expected:

1. - `copier copy -r v1.0.0 ...`
   - `copier update -r v1.1.0 ...`
   - `copier update -r v2.0.0 ...`
2. - `copier copy -r v2.0.0 ...`

## v2.0.0 (update from parent v1.0.0 to v2.0.0)

Questions: same as parent template v2.0.0, plus age.

### Files

Same as v1.0.0.

### How inheritance is implemented

Parent target version is changed in `copier.yml`:
```
parent_template_version:
  type: str
  when: false
  default: "v2.0.0"
```

`parent.yml` needs to be updated to the content of v2.0.0, which includes the new question `favorite_color`.
This requires to change:
```
parent_answers_cmd:
  type: str
  when: false
  default: "-d name='{{name}}' -d favorite_animal='{{favorite_animal}}' -d favorite_color='{{favorite_color}}'"
```
in `copier.yml`.

Added a migration to update the parent template:
```
_migrations:
  - version: "v2.0.0"
    command: "git stash && copier update -r {{parent_template_version}} -a .copier-answers-parent.yml {{parent_answers_cmd}} {{parent_excluded_files_cmd}} && git stash pop"
```

Notice the `git stash` commands. These are necessary to avoid the error
```
> Running task 1 of 1: copier update -r v2.0.0 -a .copier-answers-parent.yml -d name='NAME' -d favorite_animal='ANIMAL' -x 'user.md'
Destination repository is dirty; cannot continue. Please commit or stash your local changes and retry.
Task "copier update -r v2.0.0 -a .copier-answers-parent.yml -d name='NAME' -d favorite_animal='ANIMAL' -x 'user.md'" returned non-zero exit status 1.
```
If `copier` would support natively inheritance, this could be avoided.

## v1.1.0

Questions: same as v1.0.0

### Files

Same as v1.0.0, plus:

- `qr-code`

## v1.0.0

Questions: same as parent template v1.0.0, plus age.

### Files

- `user.md` (overwrites parent file)
  ```
  # {{name}}

  This is the file of the user {{name}}.

  {{name}} is {{age}} years old.
  ```
- `summary.md`
  ```
  # Summary

  {{name}} is {{age}} years old and its favorite animal is {{favorite_animal}}.
  ```

### How inheritance is implemented

Parent `copier.yml` is stored as `parent.yml`, and included in `copier.yml` using
```
---

!include parent.yml

---
```
This is done to import the same questions and settings from the parent, avoiding repetition and possible clashes (e.g. if `_min_copier_version` is different).
If needed, some parts (e.g. unwanted `_subdirectory`, or migrations) could be overridden.

Added also
```
parent_template_url:
  type: str
  when: false
  default: "https://github.com/francesco086/copier-inheritance-via-task-and-migration-demo-parent.git"

parent_template_version:
  type: str
  when: false
  default: "v1.0.0"

parent_template_answer_file:
  type: str
  when: false
  default: ".copier-answers-parent.yml"

parent_excluded_files_cmd:
  type: str
  when: false
  default: "-x 'user.md'"

parent_answers_cmd:
  type: str
  when: false
  default: "-d name='{{name}}' -d favorite_animal='{{favorite_animal}}'"

_tasks:
    - "echo 'The copier operation is {{ _copier_operation }}'"
    - command: "copier copy -r {{parent_template_version}} -a {{parent_template_answer_file}} {{parent_answers_cmd}} {{parent_excluded_files_cmd}} {{parent_template_url}} ."
        when: "{{ _copier_operation in ['copy', 'recopy'] }}"
```
to ship the parent template, using the right (hard-coded) version.

Notice that overlapping parent-child file (`user.md`) is excluded explicitly from the parent, as the desired behavior is that the child template overrides it.

Notice also that the task of including the parent template is executed only when the operation is `copy` or `recopy`, so it does not interfere with the update operation.

Finally, notice that the answers to the parent template are stored in `.copier-answers-parent.yml`, which will be necessary at parent's update time.