# small-orm documentation

[back to table of content](table-of-content.md)

## Chapter 13 : Working with transactions

### PersistThread class

The 'PersistThread' class represent a pile of persist and delete orders.

You can flush orders when you want. For example, persist a forum posts :
```php
// Get post
$dao = $daoFactory->get("BlogBundle", "Post");
$post = $dao->findOne(["id" => $postId], [["comments"]]);

// Create thread
$thread = new PersistThread($dao->getConnection());

// Publish comments
foreach ($post->getComments() as $comment) {
    $comment->setPublished(true);
    $thread->pushPersist($comment);
}

// Commit transaction and close
$thread->flush();
$thread->close();
```

In this case, all comments are persisted in single tcp connection.

You can also use delete order :
```php
// Get post
$dao = $daoFactory->get("BlogBundle", "Post");
$post = $dao->findOne(["id" => $postId], [["comments"]]);

// Create thread
$thread = new PersistThread($dao->getConnection());

// Delete comments
foreach ($post->getComments() as $comment) {
    $thread->pushDelete($comment);
}

// Delete post
$thread->pushDelete($post);

// Commit transaction and close
$thread->flush();
$thread->close();
```

Note the close method : It is very important to close thread at the end in order to release connection. With Swoole connector, the connection is released. If you don't, as the Swoole server use a pool of connections, you will not have remaining connections in pool over time.

### Commit and rollback

You can use sql transactions in order to avoid database corrupted :
```php
// Get post
$dao = $daoFactory->get("BlogBundle", "Post");
$post = $dao->findOne(["id" => $postId], [["comments"]]);

// Create thread
$thread = new PersistThread($dao->getConnection());

// Start transaction
$thread->startTransaction();

// Delete comments
foreach ($post->getComments() as $comment) {
    $thread->pushDelete($comment);
}

// Delete post
$thread->pushDelete($post);

// Commit transaction and close
try {
    $thread->commit();
} catch(\Exception $e) {
    $thread->rollback();
    throw $e;
}
$thread->close();
```

You can flush every time you want :
```php
// Get post
$dao = $daoFactory->get("BlogBundle", "Post");
$post = $dao->findOne(["id" => $postId], [["comments"]]);

// Create thread
$thread = new PersistThread($dao->getConnection());

// Start transaction
$thread->startTransaction();

// Delete comments
foreach ($post->getComments() as $comment) {
    if ($comment->getRemovable()) {
        $thread->pushDelete($comment);
        // Send orders to mysql
        $thread->flush();
    } else {
        // A comment is not removable => rollback transaction ans exit
        $thread->rollback();
        return;
    }
}

// Delete post
$thread->pushDelete($post);

// Commit transaction and close
try {
    $thread->commit();
} catch(\Exception $e) {
    $thread->rollback();
    throw $e;
}
$thread->close();
```

### Limitation

You can use a single 'PersistThread' object for each database. You can't push orders of models from multiple databases.