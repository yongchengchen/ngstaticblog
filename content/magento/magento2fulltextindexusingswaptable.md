<!--
Categories = ["Development", "Magento"]
Description = ""
Tags = ["Development", "Magento2"]
date = "2016-12-21T21:47:31-08:00"
title = "Magento2 Fulltext Indexer using swap table"
-->

# Magento2 Fulltext Indexer using swap table

Indexing in Magento2 is much faster than Magento1, but for fulltext indexing, still have few seconds user can search nothing.

I've developed the fulltext indexer plugin using swap table. Just index fulltext to a temporary table first, after indexing finished, swap the temporary table to the working table.

## 1. Define the plugin class

```php
<?php
namespace Your\ModuleName\Plugin\Indexer\CatalogSearch;

use Magento\Framework\Search\Request\DimensionFactory;
use Magento\Framework\Indexer\ScopeResolver\IndexScopeResolver;
use Magento\Framework\App\ResourceConnection;

class IndexerHandler {
    private $dimensionFactory;
    private $indexScopeResolver;
    private $resource;

    private $isFullReindex = false;
    private $swapDimensions;

    private $indexName = \Magento\CatalogSearch\Model\Indexer\Fulltext::INDEXER_ID;

    private $getIndexNameClosure;

    public function __construct(
        DimensionFactory $dimensionFactory,
        IndexScopeResolver $indexScopeResolver,
        ResourceConnection $resource
    ) {
        $this->dimensionFactory = $dimensionFactory;
        $this->indexScopeResolver = $indexScopeResolver;
        $this->resource = $resource;
    }

    /**
     * {@inheritdoc}
     */
    public function aroundCleanIndex($origin, $processor, $dimensions)
    {
        $this->isFullReindex = true;
        $this->swapDimensions = [];
        foreach($dimensions as $di) {
            $this->swapDimensions[] = $this->dimensionFactory->create(['name' => 'swap', 'value' => $di->getValue()]);
        }
   
        $processor($this->swapDimensions);
    }

    public function aroundSaveIndex($origin,$processor, $dimensions, $documents)
    {
        $processor($this->isFullReindex ? $this->swapDimensions : $dimensions, $documents);
        if ($this->isFullReindex) {
            $swapTableName = $this->getTableName($this->swapDimensions);
            $originTableName = $this->getTableName($dimensions);

            $tmpTable = $originTableName . '_del';
            $this->resource->getConnection()
                ->query(sprintf('DROP TABLE IF EXISTS %s;', $tmpTable));

            $this->resource->getConnection()
                ->query(sprintf('CREATE TABLE IF NOT EXISTS %s(id int);', $originTableName));

            $this->resource->getConnection()
                ->query(sprintf('RENAME TABLE %s TO %s, %s To %s;', 
                        $originTableName, 
                        $tmpTable, 
                        $swapTableName, 
                        $originTableName));
        }
    }

    private function getTableName($dimensions) {
        return $this->indexScopeResolver->resolve($this->indexName, $dimensions);
    }
}
```

## Add it to plugin settings "etc/di.xml"
```xml
    <type name="Magento\CatalogSearch\Model\Indexer\IndexerHandler">
        <plugin name="fulltext_table_swap"
                type="Your\ModuleName\Plugin\Indexer\CatalogSearch\IndexerHandler"
                sortOrder="1" />
    </type>
```

All done!

