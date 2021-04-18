# WpfPagination
A simple WPF package for paginating data.

#description
The package has a pagination viewmodel and a model that one can use to easily paginate his or her data.

#installation
// Install WpfPagination as a Cake Addin

#addin nuget:?package=WpfPagination&version=1.0.0

// Install WpfPagination as a Cake Tool

#tool nuget:?package=WpfPagination&version=1.0.0

#usage
Below is a sample viewmodel in for your app that show how to use the package




using System.Collections.ObjectModel;
using System.ComponentModel;
using System.Runtime.CompilerServices;
using ui.models;
using WpfPagination.pg;

namespace ui.viewmodels
{
    /// <summary>
    /// this is a main view model to bind to the mainwindow
    /// </summary>
    public class MainVM : INotifyPropertyChanged
    {
        


        /// <summary>
        /// Observable collection to hold a list of all student
        /// </summary>
        private ObservableCollection<Student> _students;

        public ObservableCollection<Student> Students
        {
            get { return _students; }
            set { _students = value; OnPropertyChanged(); }
        }


        /// <summary>
        /// Observablecollection to hold a list of the student will be showing 
        /// to be determined by the per page and total items property of our Pagination Model.
        /// </summary>
        private ObservableCollection<Student> _students_to_display;

        

        public ObservableCollection<Student> StudentToDisplay
        {
            get { return _students_to_display; }
            set { _students_to_display = value; OnPropertyChanged(); }
        }




        /// <summary>
        /// a dummy method to add student to the obsevablecollection here on top.
        /// </summary>
        public void LoadStudents()
        {
            //we will add 100 students with a for loop
            for (int i = 0; i < 100; i++)
            {
                Students.Add(new Student { FullName = $"Stundet {i}" });
            }
        }



        /// <summary>
        /// To use our pagination, 
        /// you will require to bring in the PgVM - PaginationVM class of our package and PgModel
        /// </summary>
        public PgVM PaginationViewModel { get; set; }
        public PgModel PaginationModel { get; set; }




        /// <summary>
        /// constructor to initialize our data and our pagination classes
        /// </summary>
        public MainVM()
        {
            //initialize our list of students
            Students = new ObservableCollection<Student>();
            //initialize a list of students
            LoadStudents();
            //create pagination view model
            PaginationViewModel = new PgVM();
            //create pagination model
            //Pagination model takes 2 parameters, per page and the total number of items.
            //For this example, i will put 10 as per page and the total number of our students as the total number of items
            //the model will compute the number of pages from this by simply dividing the total items with per page.
            PaginationModel = new PgModel(10, Students.Count);
            //after creating our pagination view model and and pagination model, 
            // we need to seed our pagination view model so as to compute and create bindable figures to our ui
            PaginationViewModel.seed(PaginationModel);
            //after iniatializing all that, we now need to create the list that we are going to display
            //to do so, i will create a method knowns as Process page to do that for me and call it here
            ProcessPage();
            //we also need to listen to change of CurrentPage and PerPage since the they are binded to the pagination view and update the studenttodisplay list
            PaginationModel.PropertyChanged += PaginationModel_PropertyChanged;
        }

        /// <summary>
        /// To call processpage method each time the current page or per page changes
        /// </summary>
        /// <param name="sender"></param>
        /// <param name="e"></param>
        private void PaginationModel_PropertyChanged(object sender, System.ComponentModel.PropertyChangedEventArgs e)
        {
            if (e.PropertyName == nameof(PaginationModel.CurrentPage) || e.PropertyName == nameof(PaginationModel.PerPage))
            {
                // we check first if the user inputed a current page that is more than Total pages
                if (PaginationModel.CurrentPage > PaginationModel.TotalPages)
                        PaginationModel.CurrentPage = PaginationModel.TotalPages;

                //we also check for 0 as current page
                if (PaginationModel.CurrentPage < 1)
                    PaginationModel.CurrentPage = 1;

                //we can also check if per page is more than total items and rectify that
                if (PaginationModel.PerPage > PaginationModel.TotalItems)
                    PaginationModel.PerPage = PaginationModel.TotalItems;

                // we also check for zeros
                if (PaginationModel.PerPage < 1)
                    PaginationModel.PerPage = 1;

                //if everything is okey, we process page to update data to display
                ProcessPage();
            }
        }


        /// <summary>
        /// to create the list of the student we are going to show out of the total of 100
        /// </summary>
        private void ProcessPage()
        {
            //first empty the list of students to display
            StudentToDisplay = new ObservableCollection<Student>();
            //now to paginate
            int start_count = (PaginationViewModel.PgModel.CurrentPage - 1) * PaginationViewModel.PgModel.PerPage;
            //now we loop our list of all students and take the ones we want
            for (int i = start_count; i < start_count + PaginationViewModel.PgModel.PerPage; i++)
            {
                if (i < Students.Count)
                {
                    StudentToDisplay.Add(Students[i]);
                }
            }
        }


        /// <summary>
        /// inotifychange interface implementation
        /// </summary>
        public event PropertyChangedEventHandler PropertyChanged;
        public void OnPropertyChanged([CallerMemberName] string property = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(property));
        }
    }
}



// your model


namespace ui.models
{
    public class Student
    {
        public string FullName { get; set; }
    }
}

//your view



<Window x:Class="ui.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:ui" xmlns:pg="clr-namespace:WpfPagination.pg;assembly=WpfPagination"
        mc:Ignorable="d"
        xmlns:viewmodels="clr-namespace:ui.viewmodels"
        Title="Pagination Example" Height="650" Width="1200">
    <Window.Resources>
        <viewmodels:MainVM x:Key="MainViewModelToBeOurDataContext" />
    </Window.Resources>
    <Grid DataContext="{StaticResource MainViewModelToBeOurDataContext}">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
        
        </Grid.RowDefinitions>
        <TextBlock Text="Our Paginated Student List" FontSize="20" FontWeight="Bold" HorizontalAlignment="Center" Margin="20" />
        <ScrollViewer Grid.Row="1" HorizontalScrollBarVisibility="Auto" VerticalScrollBarVisibility="Auto" Width="700" Height="500" Margin="231,0,119,0">
            <StackPanel>
                
                <ListBox ItemsSource="{Binding StudentToDisplay}">
                    <ListBox.ItemTemplate>
                        <DataTemplate>
                            <Border Background="AliceBlue" Padding="5" Margin="3" CornerRadius="5">
                                <TextBlock Text="{Binding FullName}" />
                            </Border>
                        </DataTemplate>
                    </ListBox.ItemTemplate>
                </ListBox>
                <pg:PaginationVW Grid.Row="1" VerticalAlignment="Bottom" DataContext="{Binding PaginationViewModel}"/>
            </StackPanel>
        </ScrollViewer>
        
       
    </Grid>
</Window>


#Created by
https://thejnmedia.com/
