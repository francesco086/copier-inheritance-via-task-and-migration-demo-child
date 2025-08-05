# Copier template inheritance via task and migrations demo: Child Template

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