# sfstarter

return (
  (!normalizedFilter.disputeServiceItem ||
    data?.disputedSerVItem?.some(item =>
      item.toLowerCase().includes(normalizedFilter.disputeServiceItem.toLowerCase())
    )) &&
  (!normalizedFilter.locationOfService ||
    (Array.isArray(data?.locationOfService)
      ? data.locationOfService.some(item =>
          item.toLowerCase().includes(normalizedFilter.locationOfService.toLowerCase())
        )
      : data?.locationOfService?.toLowerCase().includes(normalizedFilter.locationOfService.toLowerCase()))) &&
  (!normalizedFilter.processType ||
    (Array.isArray(data?.processType)
      ? data.processType.some(item =>
          item.toLowerCase().includes(normalizedFilter.processType.toLowerCase())
        )
      : data?.processType?.toLowerCase().includes(normalizedFilter.processType.toLowerCase()))) &&
  (!normalizedFilter.flags ||
    (Array.isArray(data?.flags)
      ? data.flags.some(item =>
          item.toLowerCase().includes(normalizedFilter.flags.toLowerCase())
        )
      : data?.flags?.toLowerCase().includes(normalizedFilter.flags.toLowerCase())))
);


