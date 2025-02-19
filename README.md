# sfstarter

return (
  (!normalizedFilter.disputeServiceItem ||
    (Array.isArray(normalizedFilter.disputeServiceItem) &&
      normalizedFilter.disputeServiceItem.some(
        (item) => data?.disputedSerVItem?.toLowerCase() === item.toLowerCase()
      ))) &&
  (!normalizedFilter.locationOfService ||
    (Array.isArray(normalizedFilter.locationOfService) &&
      normalizedFilter.locationOfService.some(
        (item) => data?.locationOfService?.toLowerCase() === item.toLowerCase()
      ))) &&
  (!normalizedFilter.processType ||
    (Array.isArray(normalizedFilter.processType) &&
      normalizedFilter.processType.some(
        (item) => data?.processType?.toLowerCase() === item.toLowerCase()
      ))) &&
  (!normalizedFilter.flags ||
    (Array.isArray(normalizedFilter.flags) &&
      normalizedFilter.flags.some(
        (item) => data?.flags?.toLowerCase() === item.toLowerCase()
      )))
);


