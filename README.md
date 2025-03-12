rm -rf /var/www/bitrix/bitrix/cache/*
rm -rf /var/www/bitrix/bitrix/managed_cache/*
rm -rf /var/www/bitrix/bitrix/stack_cache/*
rm -rf /var/www/bitrix/upload/tmp/*


find /var/www/bitrix -xdev -type d -printf '%d\t%p\n' | sort -nr | head -20
