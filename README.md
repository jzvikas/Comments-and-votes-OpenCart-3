:warning: The extension works but may need further improvement. Send [me](mail2andyk@gmail.com) a message if something goes wrong.

## Purpose
Simple advanced features for comments on the product page or any inforational or blog page of an OpenCart 3.x online store. These features include likes for comments and multilevel comments (up to 3d level in this aproach, customer's reply on a particular third-level-comment will display on the same 3d level after that comment).

## When it's important?
Very often for commercial products. You can see votes, likes or dislikes, share links almost on every modern commercial website that includes customers comments somewhere. Even these comments may be organised in multilevel structure.

## Prerequisites before usage
* Upload files ... and insert code between commented 3 dots ("...") into corresponding file. Please, be careful.
* You have to create table(s) in MySQL database (further description).
* If You gonna insert parts of code for TLT Blog, please, visit the [official page](https://www.opencart.com/index.php?route=marketplace/extension/info&extension_id=24602) of this free module. (I used TLT Blog for Opencart 3.0.x. license_tltblog.txt is in the root directory). catalog/controller/extension/tltblog/tltblog.php contains realized multilevel comments building approach.
* Test your result. CSS styles on your page may not dislay correctly, especially when your theme is not "default".

### Prepare MySQL DB
Firstly I'd like to demonstrate how to prepare table for product page.
1) OpenCart database should have tables called `oc_review`. There you need to add 2 columns to store.

```
ALTER TABLE `your_database`.`oc_review` ADD COLUMN `approval` TINYINT(1) NOT NULL
ALTER TABLE `your_database`.`oc_review` ADD COLUMN `disapproval` TINYINT(1) NOT NULL
```
2) Create new table `oc_review_approval` (it will have many-to-many relationship with `oc_review`):
```
CREATE TABLE `your_database`.`oc_review_approval` (
    customer_id int NOT NULL,
    review_id int NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES oc_review(customer_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    approval BOOLEAN NOT NULL,
    FOREIGN KEY (review_id) REFERENCES oc_review(review_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    PRIMARY KEY (customer_id, review_id)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

Also, I show an example of preparing tables for reviews on TLT Blog pages. By default the free version of this blog extension doesn't provide reviews functionality. So, if you need this, you have to create 2 tables.

```
CREATE TABLE `your_database`.`oc_blog_review` (
  `review_id` int(11) NOT NULL AUTO_INCREMENT,
  `tltblog_id` int(11) NOT NULL,
  `customer_id` int(11) NOT NULL,
  `depth` tinyint(1) NOT NULL,
  `related` tinyint(1) NOT NULL DEFAULT '0',
  `author` varchar(64) NOT NULL,
  `text` text NOT NULL,
  `status` tinyint(1) NOT NULL DEFAULT '0',
  `date_added` datetime NOT NULL,
  `date_modified` datetime,
  `approval` tinyint(1) NOT NULL,
  `disapproval` tinyint(1) NOT NULL,
  PRIMARY KEY (`review_id`),
  KEY `tltblog_id` (`tltblog_id`)
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

CREATE TABLE `your_database`.`oc_blog_review_approval` (
    `customer_id` int NOT NULL,
    `review_id` int NOT NULL,
    FOREIGN KEY (`customer_id`) REFERENCES `oc_blog_review`(`customer_id`)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    `approval` BOOLEAN NOT NULL,
    FOREIGN KEY (`review_id`) REFERENCES `oc_blog_review`(`review_id`)
        ON DELETE CASCADE
        ON UPDATE CASCADE,
    PRIMARY KEY (`customer_id`, `review_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

### Tests
For product page:
```
SELECT * FROM `oc_review` WHERE review_id = "1" // in adminer
```
Count number of columns by specific criteria: 
```
SELECT COUNT(*) FROM `your_database`.COLUMNS WHERE TABLE_NAME='oc_review' AND column_name='review_id'
```

Insert review into `oc_blog_review`:
```
INSERT INTO `oc_blog_review` SET author = 'Andy Kre', customer_id = '1', tltblog_id = '2', text = 'Yeah)', depth = '2', date_added = NOW();
```

Get Review from `oc_blog_review`:
```
SELECT r.review_id, r.author, r.depth, r.text, r.approval, r.disapproval, b.tltblog_id, r.date_added FROM `oc_blog_review` r LEFT JOIN `oc_tltblog` b ON (r.tltblog_id = b.tltblog_id) LEFT JOIN `oc_tltblog_description` bd ON (b.tltblog_id = bd.tltblog_id) WHERE b.tltblog_id = '2' AND b.status = '1' AND r.status = '0' AND bd.language_id = '2' ORDER BY r.date_added DESC;
```