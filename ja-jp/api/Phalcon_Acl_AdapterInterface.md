---
layout: article
language: 'ja-jp'
version: '4.0'
title: 'Phalcon\Acl\AdapterInterface'
---
# Interface **Phalcon\Acl\AdapterInterface**

[Source on Github](https://github.com/phalcon/cphalcon/tree/v{{ page.version }}.0/phalcon/acl/adapterinterface.zep)

## メソッド

abstract public **setDefaultAction** (*mixed* $defaultAccess)

デフォルトのアクセスレベル (Phalcon\Acl::ALLOW または Phalcon\Acl::DENY)をセットします。

abstract public **getDefaultAction** ()

デフォルトのACLアクセスレベルを返します

abstract public **setNoArgumentsDefaultAction** (*mixed* $defaultAccess)

isAllowedアクションに引数が指定されていなかった際のデフォルトのアクセスレベル(Phalcon\Acl::ALLOW または Phalcon\Acl::DENY)を設定します。ただしaccessKeyのfuncは存在しているものとします。

abstract public **getNoArgumentsDefaultAction** ()

isAllowedアクションに引数が指定されていなかった際のデフォルトのアクセスレベルを返します。ただしaccessKeyのfuncは存在しているものとします。

abstract public **addOperation** (*mixed* $operation, [*mixed* $accessInherits])

Adds a operation to the ACL list. Second parameter lets to inherit access data from other existing operation

abstract public **addInherit** (*mixed* $operationName, *mixed* $operationToInherit)

Do a operation inherit from another existing operation

abstract public **isOperation** (*mixed* $operationName)

Check whether operation exist in the operations list

abstract public **isSubject** (*mixed* $subjectName)

Check whether subject exist in the subjects list

abstract public **addSubject** (*mixed* $subjectObject, *mixed* $accessList)

Adds a subject to the ACL list Access names can be a particular action, by example search, update, delete, etc or a list of them

abstract public **addSubjectAccess** (*mixed* $subjectName, *mixed* $accessList)

Adds access to subjects

abstract public **dropSubjectAccess** (*mixed* $subjectName, *mixed* $accessList)

Removes an access from a subject

abstract public **allow** (*mixed* $operationName, *mixed* $subjectName, *mixed* $access, [*mixed* $func])

Allow access to a operation on a subject

abstract public **deny** (*mixed* $operationName, *mixed* $subjectName, *mixed* $access, [*mixed* $func])

Deny access to a operation on a subject

abstract public **isAllowed** (*mixed* $operationName, *mixed* $subjectName, *mixed* $access, [*array* $parameters])

Check whether a operation is allowed to access an action from a subject

abstract public **getActiveOperation** ()

Returns the operation which the list is checking if it's allowed to certain subject/access

abstract public **getActiveSubject** ()

Returns the subject which the list is checking if some operation can access it

abstract public **getActiveAccess** ()

Returns the access which the list is checking if some operation can access it

abstract public **getOperations** ()

Return an array with every operation registered in the list

abstract public **getSubjects** ()

Return an array with every subject registered in the list