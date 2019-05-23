---
group: php-developer-guide
title: Handling outdated in-memory object states
functional_areas:
  - Configuration
  - System
  - Setup
---

When Magento is launched, the store's configuration is loaded and copied into memory. Magento uses that copy in memory for queue message transactions. When the store's configuration is updated in the admin, the copy in memory does not automatically refresh resulting in an outdated in-memory object state.

To ensure the copy in memory is re-instantiated after an update, use the `PoisonPill` interface available in the MessageQueue module. Before a new message is processed, the implementation of the `PoisonPillCompareInterface` compares the version of the in-memory copy to the latest in the database. If the versions are different, the in-memory copy is destroyed and a new copy is created when the next `cron:run` job executes.

{: .bs-callout .bs-callout-info }
If the `PoisonPill` interface determines the copy in memory needs to be re-instantiated and you are running the `cron:consumers_runner` command, Magento will automatically restart. Otherwise, you will need to manually restart after the copy is created.

The `PoisonPill` interface includes the following:

Interface | Description
--- | ---
`PoisonPillCompareInterface` | Compares the in-memory copy to the database.
`PoisonPillPutInterface` | Location where you want to begin the compare.
`PoisonPillReadInterface` | Result of the compare so you can process as needed.

**Example**

The following example demonstrates how to implement the `PoisonPill` interface.

``` php
public function afterSave()
    {
        if ($this->isObjectNew()) {
            $this->_storeManager->reinitStores();
        }
        $this->pillPut->put();
        return parent::afterSave();
    }
```