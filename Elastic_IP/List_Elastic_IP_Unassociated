import boto3
import pandas as pd
 
# Initialize a session with a specific AWS profile
session = boto3.session.Session(profile_name="default")
 
def get_all_regions():
    """
    Fetches all available AWS regions.
 
    Returns:
        list: A list of AWS region names.
    """
    ec2 = session.client('ec2', region_name='us-east-1')  # Use any region to fetch available regions
    response = ec2.describe_regions()
    return [region['RegionName'] for region in response['Regions']]
 
def list_unattached_eips_for_region(region):
    """
    Lists unattached Elastic IPs for a specific region.
 
    Args:
        region (str): The AWS region to check.
 
    Returns:
        list: A list of dictionaries with details of unattached EIPs.
    """
    ec2 = session.client('ec2', region_name=region)
 
    try:
        # Retrieve all Elastic IP addresses for the specified region
        response = ec2.describe_addresses()
        addresses = response['Addresses']
 
        # Filter EIPs that do not have an AssociationId (i.e., unattached EIPs)
        unattached_eips = [
            {
                'Region': region,
                'PublicIp': addr.get('PublicIp'),
                'AllocationId': addr.get('AllocationId'),
                'Domain': addr.get('Domain'),
                'NetworkInterfaceId': addr.get('NetworkInterfaceId', 'N/A'),
                'AssociationId': addr.get('AssociationId', 'N/A')  # 'N/A' for unattached EIPs
            }
            for addr in addresses
            if 'AssociationId' not in addr  # Check if the EIP does not have an AssociationId
        ]
        return unattached_eips
 
    except Exception as e:
        print(f"Error fetching Elastic IP addresses from region {region}: {e}")
        return []
 
def list_unattached_eips_to_csv(filename):
    """
    Lists all unattached Elastic IPs from all regions and saves them to a CSV file.
 
    Args:
        filename (str): The name of the CSV file to save the data.
    """
    regions = get_all_regions()
    all_unattached_eips = []
 
    for region in regions:
        print(f"Checking region {region}...")
        unattached_eips = list_unattached_eips_for_region(region)
        all_unattached_eips.extend(unattached_eips)
 
    if all_unattached_eips:
        # Convert list of dictionaries to DataFrame
        df = pd.DataFrame(all_unattached_eips)
        # Save DataFrame to CSV
        df.to_csv(filename, index=False)
        print(f"Unattached Elastic IPs have been saved to {filename}.")
    else:
        print("All Elastic IP addresses are currently associated with resources in all regions.")
 
if __name__ == "__main__":
    # Specify the CSV filename
    csv_filename = 'unattached_eips_all_regions.csv'
    list_unattached_eips_to_csv(csv_filename)
