









































//*********************************************************
//Update Methods
 private void UpdateProducts()
        {
            try
            {
                noResultTxb.Visibility = Visibility.Hidden;
                productsLView.Visibility = Visibility.Visible;
                var products = ProjectManager.Context.Product.ToList();
                if (manufacturerBox.SelectedIndex > 0)
                    products = (manufacturerBox.SelectedItem as Manufacturer).Product.ToList();
                products = products.Where(p => p.Title.ToLower().Contains(searchingBox.Text.ToLower()) ||
                        (p.Description != null && p.Description.ToLower().Contains(searchingBox.Text.ToLower()))).ToList();
                if (string.IsNullOrWhiteSpace(searchingBox.Text))
                {
                    if (manufacturerBox.SelectedIndex > 0)
                        products = (manufacturerBox.SelectedItem as Manufacturer).Product.ToList();
                    prods = new ObservableCollection<Product>(products);
                    productsLView.ItemsSource = prods;
                    UpdateCollection();
                    return;
                }
                prods = new ObservableCollection<Product>(products);
                productsLView.ItemsSource = prods;
                UpdateCollection();
            }
            catch (Exception ex)
            {
                ProjectManager.ShowError(ex.Message);
            }
        }
//*********************************************************
var allManuf = ProjectManager.Context.Manufacturer.ToList();
            allManuf.Insert(0, new Manufacturer
            {
                Name = "Все производители"
            });
            manufacturerBox.ItemsSource = allManuf;
            manufacturerBox.SelectedIndex = 0;
//*********************************************************
private void Sorting()
        {
           var products = productsLView.ItemsSource.Cast<Product>();
            switch (sortBox.SelectedIndex)
            {
                default:
                    productsLView.ItemsSource = products.OrderBy(p => p.ID).ToList();
                    break;
                case 1:
                    productsLView.ItemsSource = products.OrderBy(p => p.Cost).ToList();
                    break;
                case 2:
                    productsLView.ItemsSource = products.OrderByDescending(p => p.Cost).ToList();
                    break;
            }
	}
//*********************************************************
private void Delete_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                Product selectedProduct = productsLView.SelectedItem as Product;
                if (selectedProduct.ProductSale.Count != 0)
                {
                    ProjectManager.ShowWarning("Нельзя удалить уже проданный товар!");
                    return;
                }

                MessageBoxResult result = ProjectManager.ShowQuestion("Вы действительно хотите удалить выбранный товар?");

                if (result == MessageBoxResult.No)
                    return;

                selectedProduct.Product1.Clear();
                if (!string.IsNullOrWhiteSpace(selectedProduct.MainImagePath))
                    File.Delete(selectedProduct.MainImagePath);
                ProjectManager.Context.Product.Remove(selectedProduct);
                ProjectManager.Context.SaveChanges();
                ProjectManager.ShowInformation("Товар и информация по прикрепленным товарам успешно удалена!");
                UpdateProducts();
            }
            catch (Exception ex)
            {
                ProjectManager.ShowError(ex.Message);
            }
        }
//*********************************************************
private void UpdateCollection()
        {
            try
            {
                statsBlock.Text = $"Показано {productsLView.Items.Count} из {ProjectManager.Context.Product.Count()} элементов";
                NoResultReply();
            }
            catch (Exception ex)
            {
                ProjectManager.ShowError(ex.Message);
                statsBlock.Text = string.Empty;
            }
        }
//*********************************************************
private void NoResultReply()
        {
            Dispatcher.BeginInvoke(new Action(() =>
            {
                if (productsLView.Items.Count <= 0)
                {
                    noResultTxb.Visibility = Visibility.Visible;
                    productsLView.Visibility = Visibility.Hidden;
                }
            }), System.Windows.Threading.DispatcherPriority.Background);

        }
//*********************************************************
public static class ProjectManager
    {
        public static AM_КравецEntities Context = new AM_КравецEntities();

        public static int UserRole { get; set; }
        public static Frame MainFrame { get; set; }
        public static bool IsNavigationBlocked { get; set; }

        public static MessageBoxResult ShowQuestion(string a)
        {
            return MessageBox.Show(a, "Подтверждение", MessageBoxButton.YesNo, MessageBoxImage.Question);
        }
        public static MessageBoxResult ShowError(string message)
        {
            return MessageBox.Show(message, "Ошибка!", MessageBoxButton.OK, MessageBoxImage.Question);
        }
        public static MessageBoxResult ShowWarning(string message)
        {
            return MessageBox.Show(message, "Предупреждение", MessageBoxButton.OK, MessageBoxImage.Question);
        }
        public static MessageBoxResult ShowInformation(string message)
        {
            return MessageBox.Show(message, "Уведомление", MessageBoxButton.OK, MessageBoxImage.Question);
        }

    }
//*********************************************************
private void UploadImage_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                if (!fileDialog.ShowDialog().Value)
                    return;

                if (new FileInfo(fileDialog.FileName).Length > 1024 * 1024 * 2)
                {
                    ProjectManager.ShowWarning("Размер изображения не должен превышать 2 МБ!");
                    fileDialog.FileName = null;
                    return;
                }

                mainImage.Source = new BitmapImage(new Uri(fileDialog.FileName));
            }
            catch (Exception ex)
            {
                ProjectManager.ShowError(ex.Message);
                ProjectManager.MainFrame.Navigate(new ProductPage());
            }
        }
//*********************************************************
private void SaveImage_Click(object sender, RoutedEventArgs e)
        {
            if (!string.IsNullOrWhiteSpace(fileDialog.FileName))
                {
                    if (!string.IsNullOrWhiteSpace(Product.MainImagePath))
                        File.Delete(Product.MainImagePath);

                    string format = fileDialog.FileName.Split('.').LastOrDefault();
                    string photoPath = $@"Товарышколы\photo_{Product.ID}.{format}";
                    File.Copy(fileDialog.FileName, photoPath, true);
                    Product.MainImagePath = photoPath;
                    ProjectManager.Context.SaveChanges();
                }
        }
//*********************************************************
//Image things
OpenFileDialog fileDialog = new OpenFileDialog()
        {
            CheckFileExists = true,
            CheckPathExists = true,
            Filter = "Файлы изображений|*.jpg, *.jpeg, *.png|Все файлы|*.*"
        };
        string oldMainImagePath;
//*********************************************************
	public class ImageConverter : IValueConverter
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value == null || string.IsNullOrWhiteSpace(value.ToString()))
                return new BitmapImage(new Uri("pack://application:,,,/Resources/picture.png"));

            return new BitmapImage(new Uri(value.ToString(), UriKind.RelativeOrAbsolute));
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
//*********************************************************
    public class DiscountTextDecorationConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            if (value != null && double.TryParse(value.ToString(), out double discount) && discount > 0)
                return TextDecorations.Strikethrough;

            return null;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
//*********************************************************
//Image Converter
public class FromStringToImageConverter : IValueConverter
    {
        private string imageDirectory = Directory.GetCurrentDirectory();
        public string ImageDirectory
        {
            get { return imageDirectory; }
            set { imageDirectory = value; }
        }

        public object Convert(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            if ((string)value != null)
            {
                string imagePath = Path.Combine(ImageDirectory, (string)value);
                if (File.Exists(imagePath))
                {
                    var bitmap = new BitmapImage();
                    bitmap.BeginInit();
                    bitmap.UriSource = new Uri(imagePath);
                    bitmap.CacheOption = BitmapCacheOption.OnLoad;
                    bitmap.CreateOptions = BitmapCreateOptions.IgnoreImageCache;
                    bitmap.EndInit();
                    return bitmap;
                }
                return null;
            }
            return null;
        }

        public object ConvertBack(object value, Type targetType, object parameter, System.Globalization.CultureInfo culture)
        {
            throw new NotSupportedException("The method or operation is not implemented.");
        }

    }
//*********************************************************
        public void cptcha()
        {
            cap.Visibility = kapcha.Visibility = namkap.Visibility = Visibility.Visible;
            cap.Text = kapcha.Text = parBox.Password = "";
            string a = "ABCDEFGHIJKLMNOPORSTUMWXYZ1234567890";
            Random b = new Random();
            for (int i = 0; i < 6; i++)
                cap.Text += a[b.Next(0, a.Length)];
        }
//*********************************************************
<StackPanel Orientation="Horizontal">
            <TextBlock x:Name="namkap" Text="Введите капчу: "/>
            <TextBlock x:Name="cap" Grid.Column="2" Grid.Row="1" TextDecorations="Strikethrough" RenderTransformOrigin="0.5,0.5" > >
                <TextBlock.RenderTransform>
                    <TransformGroup>
                        <ScaleTransform/>
                        <SkewTransform/>
                        <RotateTransform Angle="4"/>
                        <TranslateTransform/>
                    </TransformGroup>
                </TextBlock.RenderTransform>
            </TextBlock>
            <TextBox x:Name="kapcha" Width="150"/>
        </StackPanel>
//*********************************************************
//Layout quad
	<TextBlock x:Name="noResultTxb" Panel.ZIndex="1" Margin="10,10,210,9.6" TextAlignment="Center" Padding="200" Text="Нет результатов поиска"/>
        <TextBlock x:Name="statsBlock" Width="auto" Margin="10 0 0 5" HorizontalAlignment="Left" VerticalAlignment="Bottom"/>
        <ListView x:Name="productsLView" Margin="10,10,210,29.6" SelectionMode="Single"
                  ScrollViewer.HorizontalScrollBarVisibility="Disabled" HorizontalContentAlignment="Center" SelectionChanged="productsLView_SelectionChanged">
            <ListView.ItemsPanel>
                <ItemsPanelTemplate>
                    <WrapPanel Orientation="Horizontal" HorizontalAlignment="Center"/>
                </ItemsPanelTemplate>
            </ListView.ItemsPanel>
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Border Name="border" ToolTip="{Binding Title}" Margin="5" BorderBrush="Gray" CornerRadius="3" BorderThickness="1">
                        <StackPanel Width="235" Height="250" HorizontalAlignment="Center">
                            <Image Width="150" Height="150" Source="{Binding MainImagePath, Converter={StaticResource FromStringToImage}, TargetNullValue={StaticResource ImageNull}}"
                                   Stretch="Uniform" HorizontalAlignment="Center" Margin="5"/>
                            <TextBlock Text="{Binding Cost, StringFormat={}{0:N2}руб.}" VerticalAlignment="Center" HorizontalAlignment="Center" TextWrapping="Wrap" TextAlignment="Center"
                                       Margin="5 5" FontSize="12"/>
                            <TextBlock Text="{Binding Title}" Margin="5 5 5 15" HorizontalAlignment="Center" FontSize="12" TextWrapping="NoWrap"
                                       TextTrimming="WordEllipsis" TextAlignment="Center" FontWeight="Bold"/>
                            <TextBlock Text="{Binding IsActive, Converter={StaticResource FromBoolToStringActive}}" FontSize="10" HorizontalAlignment="Center" VerticalAlignment="Center"/>
                        </StackPanel>
                    </Border>
                    <DataTemplate.Triggers>
                        <DataTrigger Binding="{Binding IsActive}" Value="False">
                            <Setter TargetName="border" Property="Background" Value="LightGray"/>
                        </DataTrigger>
                    </DataTemplate.Triggers>
                </DataTemplate>
            </ListView.ItemTemplate>
            <ListView.ItemContainerStyle>
                <Style TargetType="{x:Type ListViewItem}">
                    <Setter Property="ToolTip" Value="{Binding Title}"/>
                </Style>
            </ListView.ItemContainerStyle>
        </ListView>
//*********************************************************
//Layout stack
<Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="auto"/>
            <RowDefinition/>
            <RowDefinition Height="auto"/>
        </Grid.RowDefinitions>
        <ListView x:Name="LVMaterial" ItemsSource="{Binding}" Grid.Row="1" HorizontalContentAlignment="Stretch">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Border Margin="3" BorderBrush="Black" BorderThickness="1">
                        <Grid>
                            <Grid.ColumnDefinitions>
                                <ColumnDefinition Width="*" />
                                <ColumnDefinition Width="*" />
                                <ColumnDefinition Width="10*" />
                                <ColumnDefinition Width="*" />
                            </Grid.ColumnDefinitions>
                            <Image Grid.Column="0" Width="100" Height="60" Margin="10,5,0,5" Source="{Binding Image, Converter={StaticResource StringToImage}, TargetNullValue={StaticResource ImageNull}}"
                                Stretch="Uniform" />
                            <StackPanel Grid.Column="1" Margin="15,10,0,0">
                                <DockPanel>
                                    <TextBlock FontSize="14">
                                        <TextBlock.Style>
                                                <Style TargetType="{x:Type TextBlock}">
                                                    <Setter Property="Text">
                                                        <Setter.Value>
                                                            <MultiBinding StringFormat="{}{0} | {1}">
                                                                <Binding Path="MaterialType.Title" />
                                                                <Binding Path="Title" />
                                                            </MultiBinding>
                                                        </Setter.Value>
                                                    </Setter>
                                                </Style>
                                            </TextBlock.Style>
                                    </TextBlock>
                                </DockPanel>
                                <DockPanel Margin="0,3,0,0">
                                    <TextBlock Text="Минимальное количество: " />
                                    <TextBlock Text="{Binding MinCount, StringFormat={} {0} шт}" />
                                </DockPanel>
                                <DockPanel Margin="0,3,0,0">
                                    <TextBlock MinHeight="45" Width="320" TextWrapping="Wrap">
                                        <Run FontWeight="Bold" Text="Поставщики:"/> <Run Text="{Binding Supplier, Converter={StaticResource SuppConverter}}"/>
                                    </TextBlock>
                                </DockPanel>
                            </StackPanel>

                            <StackPanel Grid.Column="3" Margin="0,13,0,0" HorizontalAlignment="Right" Orientation="Horizontal">
                                <TextBlock HorizontalAlignment="Right" VerticalAlignment="Top" Text="Остаток:" />
                                <TextBlock Margin="5,0,5,0" VerticalAlignment="Top" Text="{Binding CountInStock, StringFormat={} {0} шт}" />
                            </StackPanel>
                        </Grid>
                    </Border>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
        <StackPanel x:Name="PageSelectStk" Orientation="Horizontal" Grid.Row="2" HorizontalAlignment="Right" Margin="0 0 5 0"/>
    </Grid>