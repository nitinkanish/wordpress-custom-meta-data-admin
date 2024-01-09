# wordpress-custom-meta-data-admin

  // Add the custom menu item
function custom_user_list_menu() {
    add_menu_page(
        'Schools List',        // Page title
        'Schools List',        // Menu title
        'manage_options',   // Capability required to access
        'custom-user-list', // Menu slug
        'display_user_list', // Callback function to display the page content
        'dashicons-chart-pie'
    );
}
add_action('admin_menu', 'custom_user_list_menu');

class Custom_User_List_Table extends WP_List_Table {
    function get_columns() {
        return array(
            'school_name'   => 'School Name',
            'school_count'  => 'Count',
            'school_action' => 'Action',
        );
    }
    function prepare_items() {
        global $wpdb;
        $query = "SELECT meta_value AS school_name, COUNT(*) AS school_count FROM {$wpdb->prefix}usermeta WHERE meta_key = 'school_name' GROUP BY meta_value;";
        $data = $wpdb->get_results($query);
        $columns = $this->get_columns();
        $hidden = array();
        $sortable = $this->get_sortable_columns();

        $this->_column_headers = array($columns, $hidden, $sortable);

        // Sort the data based on the sorted column and order
        usort($data, array($this, 'usort_reorder'));
        $per_page = 16; // Number of items per page
        $current_page = $this->get_pagenum();
        $total_items = count($data);
        $this->set_pagination_args(array(
            'total_items' => $total_items,
            'per_page'    => $per_page,
            'total_pages' => ceil($total_items / $per_page),
        ));
        $this->items = array_slice($data, (($current_page - 1) * $per_page), $per_page);
    }

    // Custom sorting function for 'school_name' column
    function usort_reorder($a, $b) {
        $orderby = (!empty($_REQUEST['orderby'])) ? $_REQUEST['orderby'] : 'school_name';
        $order = (!empty($_REQUEST['order'])) ? $_REQUEST['order'] : 'asc';
        switch ($orderby) {
            case 'school_name':
                $result = strcmp($a->school_name, $b->school_name);
                break;
            case 'school_count':
                $result = $a->school_count - $b->school_count;
                break;
            default:
                $result = 0;
                break;
        }
        return ($order === 'asc') ? $result : -$result;
    }

    // Make the 'school_name' column sortable
    function get_sortable_columns() {
        return array(
            'school_name' => array('school_name', false),
            'school_count' => array('school_count', false),
        );
    }

    function column_default($item, $column_name) {
        return isset($item->$column_name) ? $item->$column_name : '';
    }

    function column_school_name($item) {
        if (!empty($item->school_name)) {
            return $item->school_name;
        } else {
            return '-';
        }
    }

    function column_school_action($item) {
        return '<button schoolname="' . $item->school_name . '" class="button action_school_delete">Delete</button>';
    }

    function column_school_count($item) {
        return number_format($item->school_count); // Format the number with commas
    }
}



// Display the user list
function display_user_list() {
    // Get all users
    global $wpdb;

    $custom_user_list_table = new Custom_User_List_Table();
    $custom_user_list_table->prepare_items();
    
    echo '<div class="wrap">';
    echo '<h2>School Name Counts</h2>';
    $custom_user_list_table->display();
    echo '</div>';
    
    ?>
<script>
jQuery(document).ready(function($) {
    $('.action_school_delete').on('click', function(e) {
        var schoolName = $(this).attr('schoolname');
debugger
        // Show a confirmation popup
        var confirmDelete = confirm('Are you sure you want to delete the school: ' + schoolName + '?');

        if (confirmDelete) {
            // Send AJAX request
            $.ajax({
                type: 'POST',
                url: ajaxurl, // WordPress AJAX handler
                data: {
                    action: 'delete_school',
                    school_name: schoolName
                },
                success: function(response) {
                    // Handle the response if needed
                    console.log(response);
                    location.reload();
                }
            });
        }
    });
});
</script>


    <?php
}
// 
function delete_school_callback() {
    if (isset($_POST['school_name'])) {
        global $wpdb;

        $school_name = sanitize_text_field($_POST['school_name']);

        // Delete the school from the database
        // $wpdb->delete(
        //     $wpdb->usermeta,
        //     array('meta_key' => 'school_name', 'meta_value' => $school_name),
        //     array('%s', '%s')
        // );
        $wpdb->update(
            $wpdb->usermeta,
            array('meta_value' => ''), // Set the meta_value to empty
            array('meta_key' => 'school_name', 'meta_value' => $school_name),
            array('%s'), // Format for the new value
            array('%s', '%s') // Format for the WHERE clause
        );

        echo $school_name.'School deleted successfully.';
    }
    wp_die(); // Always use wp_die() to end the script
}
add_action('wp_ajax_delete_school', 'delete_school_callback');




Exploring and Understanding the WordPress Code for School Name Counts
WordPress is a versatile platform that allows developers to create custom functionalities through plugins and themes. In this blog post, we'll dissect and understand a piece of code that adds a custom menu to the WordPress admin dashboard. The menu displays a list of schools along with the count of users associated with each school. Additionally, it provides an option to delete a school.

Let's break down the code step by step.

Creating a Custom Menu Item
The code begins by defining a function custom_user_list_menu() that adds a custom menu item to the WordPress admin dashboard. This menu item is labeled "Schools List" and has a corresponding page where the list of schools will be displayed. The function utilizes the add_menu_page function to achieve this.

php
Copy code
add_action('admin_menu', 'custom_user_list_menu');
The add_action function hooks into the admin_menu action, triggering the execution of custom_user_list_menu().

Custom User List Table Class
The code introduces a class named Custom_User_List_Table that extends WP_List_Table. This class is responsible for rendering the table displaying the list of schools and their associated user counts.

Columns Definition
The get_columns() method defines the columns for the table, including "School Name," "Count," and "Action."

Data Preparation
The prepare_items() method queries the WordPress database using the global $wpdb object to fetch school names and their respective user counts. It then prepares the data for display, considering pagination.

Sorting
The class provides a custom sorting function (usort_reorder) for the "School Name" and "Count" columns. It allows users to sort the table based on these columns.

Display Callbacks
The class defines methods like column_default(), column_school_name(), column_school_action(), and column_school_count() to customize the display of data in each column.

Displaying the User List
The display_user_list() function creates an instance of Custom_User_List_Table, prepares the items, and displays the table on a WordPress admin page labeled "School Name Counts."

JavaScript for School Deletion
A JavaScript script is included in the code, leveraging jQuery. It adds a click event listener to the "Delete" button in each row. When clicked, it prompts the user for confirmation and, if confirmed, triggers an AJAX request to delete the selected school.

AJAX Request Handling
The code includes a function delete_school_callback() to handle the AJAX request. It sanitizes the school name, updates the database to remove the association of users with the deleted school, and responds with a success message.

Hooking into WordPress AJAX
The last line of code hooks the delete_school_callback() function into the WordPress AJAX system, allowing it to be triggered when an AJAX request with the action 'delete_school' is received.

In summary, this WordPress code provides a comprehensive solution for managing and displaying school name counts within the admin dashboard. It incorporates database queries, custom table rendering, sorting functionalities, and AJAX-powered school deletion.
